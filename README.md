An API framework for Django where each layer hides complexity—and reveals simplicity.

Only the latest. Only the essential. No compromises.

- Django 5.2+ (LTS) only
- Pydantic v2 only
- msgspec 0.19+ only
- uv as package manager
- No legacy version support. We don’t carry the past—we build the future.

**django-matryoshka** is not a replacement for Django. It’s its ideal API layer.
You still use ORM, authentication, middleware, signals—everything that makes Django reliable.
But now you interact with it through a modern, type-annotated, testable, and self-documenting interface.

It all starts with simplicity. One endpoint—two lines of code:

```python
from matryoshka import MatryoshkaAPI
from .models import Item

api = MatryoshkaAPI()

@api.get("/items/{id}")
def get_item(request, id: int):
    return Item.objects.get(id=id)

# urls.py
urlpatterns = [path("api/", api.urls)]
```

That’s enough. You get:
— Validation of `id` as an integer (422 error on violation),
— Automatic 404 if the object is not found,
— OpenAPI documentation at `/api/docs`,
— A test-ready endpoint.

No `INSTALLED_APPS`. No migrations. No templates. No boilerplate.

Two ways to write your API — choose the right layer.

## Functions - for simplicity
When logic is straightforward, don’t overcomplicate it. Just write a function with typed arguments and a return type. Types become validation schemas, serialization rules, and OpenAPI definitions.

```python
@api.post("/items")
def create_item(request, payload: ItemCreate) -> ItemOut:
    return Item.objects.create(**payload.dump())
```

Both **Pydantic v2** (`BaseModel`) and **msgspec** (`Struct`) are supported. You choose—the framework adapts.

## Resources—for structure
When you have a cohesive set of operations (CRUD + custom actions), use the `Resource` class:

```python
from matryoshka import Resource

class ItemResource(Resource):
    model = Item
    schema_in = ItemCreate
    schema_out = ItemOut

    @action(detail=True, methods=["post"])
    def archive(self, request, id: int) -> Message:
        self.get_object(id=id).archive()
        return {"status": "archived"}
```

Routes are generated automatically:
- GET `/items/` → `list`
- POST `/items/` → `create`
- GET `/items/{id}/` → `retrieve`
- POST `/items/{id}/archive/` → `archive`

OpenAPI reflects the full structure—without a single line of configuration.

## Testing is a pleasure, not a chore

```python
def test_create_item(api_client):
    response = api_client.post_json("/items", {"name": "Laptop", "price": 999.99})
    assert response.status_code == 201
    assert response.data["name"] == "Laptop"
```

-  `post_json()` sends data as JSON with the correct Content-Type,
- `response.data` is already a parsed Python object,
- `api_client.force_authenticate(user)` enables instant authentication in tests.

OpenAPI isn’t an optional feature—it’s built in by design.
All path parameters, query strings, request bodies, and response schemas are generated automatically from type annotations and schema metadata (including constraints from `msgspec.field(metadata=...)` or Pydantic `Field`).

Validation errors are part of the specification. Custom response codes are defined via decorators.

The philosophy of django-matryoshka is like a matryoshka doll:

- The outer layer is your business logic,
- The middle layer handles routing, validation, and serialization,
- The inner layer is Django, ORM, and security.

You work only with the outer layer. Everything else is reliably hidden—but always accessible when needed.

If you’re ready to write APIs as if Django were originally designed for modern developers—welcome to django-matryoshka.
