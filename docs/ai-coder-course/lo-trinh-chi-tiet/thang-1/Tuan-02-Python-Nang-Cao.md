# 📅 TUẦN 02: Python Nâng Cao — Professional Python

> **Mục tiêu**: Async/await, type hints, Pydantic, decorators, HTTP APIs  
> **Thời lượng**: 20-25 giờ | **Output**: Weather API Wrapper + FastAPI endpoint

---

## 🎯 Sau Tuần Này Bạn Sẽ:
- [ ] Sử dụng type hints và Pydantic models
- [ ] Viết async/await code với asyncio
- [ ] Tạo decorators và context managers
- [ ] Gọi HTTP APIs (requests, httpx)
- [ ] Tạo REST API đơn giản với FastAPI
- [ ] Quản lý virtual environments (venv/uv)

---

## 📚 Khóa Học

### 🔴 Khóa chính — Udemy Business:
**[100 Days of Code™: The Complete Python Pro Bootcamp](https://ibmcsr.udemy.com/course/100-days-of-code/)**

- Giảng viên: Dr. Angela Yu | ⭐ 4.7 | 57 giờ
- **Tuần này học**: Days 30-40 (Error handling, APIs, JSON, Decorators)
  - Day 30-32: Errors, Exceptions, JSON
  - Day 33-35: APIs, HTTP requests
  - Day 36-38: Advanced Python
  - Day 39-40: Decorators, Higher-order functions

### 🟡 Bổ trợ — Udemy Business:
**[FastAPI - The Complete Course 2026](https://ibmcsr.udemy.com/course/fastapi-the-complete-course/)**

- ⭐ 4.6 | 21.5 giờ
- **Tuần này xem**: Sections 1-3 (FastAPI basics, Pydantic, first endpoint)

### 🆓 Free:
- [Real Python: Async IO](https://realpython.com/async-io-python/) — Deep dive async
- [Pydantic v2 Docs](https://docs.pydantic.dev/latest/) — Official reference
- [FastAPI Tutorial](https://fastapi.tiangolo.com/tutorial/) — Interactive docs

---

## 📅 Lịch Học Theo Ngày

### Thứ 2 — Virtual Env & Type Hints (3.5 giờ)
| Thời gian | Hoạt động | Chi tiết |
|-----------|-----------|----------|
| 1.0h | 🎬 Video | 100 Days: Error handling, JSON data |
| 1.0h | 💻 Thực hành | Tạo venv, requirements.txt, .env file |
| 1.0h | 💻 Bài tập | Refactor Todo App với type hints |
| 0.5h | 📝 Ghi chú | Type hints cheat sheet |

**Bài tập ngày:**
```python
# Bài 1: Setup project chuyên nghiệp
# Tạo:
# my_project/
# ├── .venv/
# ├── .env                  # API_KEY=xxx
# ├── .gitignore            # .venv, .env, __pycache__
# ├── requirements.txt
# ├── pyproject.toml
# └── src/
#     └── main.py

# Bài 2: Type hints practice
def calculate_average(scores: list[float]) -> float:
    """Tính trung bình, raise ValueError nếu list rỗng."""
    if not scores:
        raise ValueError("Scores list cannot be empty")
    return sum(scores) / len(scores)

def find_student(students: list[dict[str, str]], name: str) -> dict[str, str] | None:
    """Tìm student theo tên, trả về None nếu không tìm thấy."""
    pass
```

### Thứ 3 — Pydantic Models (3.5 giờ)
| Thời gian | Hoạt động | Chi tiết |
|-----------|-----------|----------|
| 1.0h | 📖 Docs | Đọc Pydantic v2 tutorial |
| 1.5h | 💻 Thực hành | Tạo models cho BookWormHub entities |
| 0.5h | 💻 Bài tập | Validation, serialization |
| 0.5h | 🧪 Test | Viết tests cho models |

**Bài tập ngày:**
```python
from pydantic import BaseModel, Field, field_validator
from datetime import datetime

class Book(BaseModel):
    title: str = Field(min_length=1, max_length=200)
    author: str
    isbn: str = Field(pattern=r'^\d{13}$')
    genre: str
    published_year: int = Field(ge=1800, le=2030)
    rating: float = Field(ge=0, le=5, default=0)
    
    @field_validator('isbn')
    @classmethod
    def validate_isbn_checksum(cls, v):
        """Validate ISBN-13 checksum"""
        # Implement ISBN-13 validation
        pass

class BookCreate(BaseModel):
    """Input model - không có id, rating"""
    title: str
    author: str
    isbn: str

class BookResponse(Book):
    """Output model - có thêm id"""
    id: int
    created_at: datetime
```

### Thứ 4 — Async/Await (4 giờ)
| Thời gian | Hoạt động | Chi tiết |
|-----------|-----------|----------|
| 1.5h | 📖 Article | Real Python async tutorial |
| 1.5h | 💻 Thực hành | asyncio basics, aiohttp, httpx async |
| 0.5h | 💻 Bài tập | Async file reader, parallel API calls |
| 0.5h | 📝 Notes | Async patterns cheat sheet |

**Bài tập ngày:**
```python
import asyncio
import httpx

# Bài 1: Async basics
async def fetch_data(url: str) -> dict:
    async with httpx.AsyncClient() as client:
        response = await client.get(url)
        return response.json()

# Bài 2: Parallel requests (so sánh tốc độ sync vs async)
async def fetch_all_parallel(urls: list[str]) -> list[dict]:
    async with httpx.AsyncClient() as client:
        tasks = [client.get(url) for url in urls]
        responses = await asyncio.gather(*tasks)
        return [r.json() for r in responses]

# Bài 3: Async generator
async def read_large_file(filepath: str):
    """Đọc file lớn từng dòng — async generator"""
    pass
```

### Thứ 5 — Decorators & Context Managers (3.5 giờ)
| Thời gian | Hoạt động | Chi tiết |
|-----------|-----------|----------|
| 1.5h | 🎬 Video | 100 Days: Decorators, Higher-order functions |
| 1.0h | 💻 Thực hành | Timer decorator, retry decorator |
| 0.5h | 💻 Bài tập | Custom context manager |
| 0.5h | 📝 Review | Decorator patterns reference |

**Bài tập ngày:**
```python
import time
import functools

# Bài 1: Timer decorator
def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{func.__name__} took {elapsed:.4f}s")
        return result
    return wrapper

# Bài 2: Retry decorator
def retry(max_attempts=3, delay=1):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            # Implement retry logic
            pass
        return wrapper
    return decorator

# Bài 3: Context manager cho database connection
from contextlib import contextmanager

@contextmanager
def db_connection(db_path: str):
    """Open DB, yield connection, ensure close"""
    pass
```

### Thứ 6 — HTTP APIs & FastAPI (3.5 giờ)
| Thời gian | Hoạt động | Chi tiết |
|-----------|-----------|----------|
| 1.0h | 🎬 Video | FastAPI Course: Sections 1-2 |
| 1.5h | 💻 Thực hành | Build Weather API wrapper |
| 0.5h | 💻 Bài tập | FastAPI CRUD endpoint |
| 0.5h | 📝 Test | Test với Swagger UI |

**Bài tập ngày:**
```python
# Bài 1: Weather API Wrapper
import httpx

class WeatherClient:
    BASE_URL = "https://api.openweathermap.org/data/2.5"
    
    def __init__(self, api_key: str):
        self.api_key = api_key
    
    async def get_weather(self, city: str) -> dict:
        """Get current weather for a city"""
        pass
    
    async def get_forecast(self, city: str, days: int = 5) -> list:
        """Get weather forecast"""
        pass

# Bài 2: FastAPI endpoint
from fastapi import FastAPI
app = FastAPI()

@app.get("/weather/{city}")
async def get_weather(city: str):
    client = WeatherClient(api_key="...")
    return await client.get_weather(city)
```

### Thứ 7 — 🏆 Project: Weather API Service (4 giờ)
**Yêu cầu:**
```
weather_service/
├── main.py           # FastAPI app
├── models.py         # Pydantic models
├── weather_client.py # Async HTTP client
├── config.py         # Settings from .env
├── requirements.txt
└── README.md         # Documentation
```

**Features:**

1. `GET /weather/{city}` — Current weather
2. `GET /forecast/{city}?days=5` — Forecast
3. Pydantic response models
4. Error handling (city not found, API error)
5. Dotenv config management

### Chủ Nhật — Review (2 giờ)

---

## ✅ Đánh Giá Cuối Tuần

| # | Tiêu chí | ✅/❌ |
|---|---------|------|
| 1 | Tạo venv, quản lý dependencies với requirements.txt | |
| 2 | Viết Pydantic model với validators | |
| 3 | Viết async function với asyncio/httpx | |
| 4 | Tạo decorator (ít nhất timer + retry) | |
| 5 | Gọi external API và parse JSON response | |
| 6 | Tạo FastAPI endpoint với Pydantic models | |
| 7 | Sử dụng .env cho secrets | |
| 8 | Weather Service chạy được trên localhost | |
| 9 | Code push lên GitHub với README | |
| 10 | Giải thích được sync vs async | |

---

> **Tiếp theo**: [Tuần 03: Git & LLM API →](Tuan-03-Git-va-LLM-API.md)
