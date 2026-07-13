# Library API - Lab 4

Sheila Mwangi
C027-01-0835/2024

## Exercise 1: Category Model

models/category.py
```python
from sqlmodel import SQLModel, Field, Relationship
from typing import List, Optional, TYPE_CHECKING

if TYPE_CHECKING:
    from models.book import Book


class Category(SQLModel, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)
    name: str = Field(unique=True, index=True, min_length=2, max_length=50)

    books: List["Book"] = Relationship(back_populates="category")
```

models/book.py additions
```python
if TYPE_CHECKING:
    from models.category import Category

class Book(SQLModel, table=True):
    category_id: Optional[int] = Field(default=None, foreign_key="category.id")
    category: Optional["Category"] = Relationship(back_populates="books")
```

main.py addition
```python
@app.post("/categories", response_model=Category)
def create_category(name: str, session: Session = Depends(get_session)):
    category = Category(name=name)
    session.add(category)
    session.commit()
    session.refresh(category)
    return category
```

## Exercise 2: Search Endpoint

```python
@app.get("/books/search", response_model=List[Book])
def search_books(
    author: Optional[str] = None,
    title: Optional[str] = None,
    session: Session = Depends(get_session)
):
    query = select(Book)

    if author:
        query = query.where(Book.author.contains(author))
    if title:
        query = query.where(Book.title.contains(title))

    return session.exec(query).all()
```

## Exercise 3: Update Endpoint

```python
@app.patch("/books/{book_id}", response_model=Book)
def update_book(
    book_id: int,
    book_update: BookUpdate,
    session: Session = Depends(get_session)
):
    book = session.get(Book, book_id)
    if not book:
        raise HTTPException(status_code=404, detail="Book not found")

    update_data = book_update.dict(exclude_unset=True)
    for field, value in update_data.items():
        setattr(book, field, value)

    book.updated_at = datetime.utcnow()
    session.commit()
    session.refresh(book)
    return book
```