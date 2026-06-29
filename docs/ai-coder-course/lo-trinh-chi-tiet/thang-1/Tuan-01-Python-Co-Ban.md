# 📅 TUẦN 01: Python Cơ Bản — Từ Zero

> **Mục tiêu**: Viết được Python cơ bản: variables, functions, OOP, file I/O  
> **Thời lượng**: 20-25 giờ | **Output**: CLI Todo App + GitHub repo đầu tiên

---

## 🎯 Sau Tuần Này Bạn Sẽ:
- [ ] Cài đặt và sử dụng Python 3.12+, VS Code
- [ ] Hiểu variables, types, control flow, loops
- [ ] Viết functions, sử dụng modules
- [ ] Làm việc với lists, dicts, sets
- [ ] Hiểu OOP cơ bản: classes, inheritance
- [ ] Đọc/ghi files (JSON, CSV, text)
- [ ] Hoàn thành CLI Todo App

---

## 📚 Khóa Học

### 🔴 Khóa chính — Udemy Business:
**[The Complete Python Bootcamp From Zero to Hero in Python](https://ibmcsr.udemy.com/course/complete-python-bootcamp/)**  

- Giảng viên: Jose Portilla (Pierian Training)
- ⭐ 4.6 | 561.720 xếp hạng | 22.5 giờ | 170 bài giảng
- **Tuần này học**: Sections 1-9 (khoảng 10 giờ video)
  - Section 1: Course Overview
  - Section 2: Python Setup
  - Section 3: Python Object and Data Structure Basics
  - Section 4: Python Comparison Operators
  - Section 5: Python Statements
  - Section 6: Methods and Functions
  - Section 7: Milestone Project 1
  - Section 8: OOP
  - Section 9: Modules and Packages

### 🟢 Bổ trợ (nếu cần thêm):
**Coursera**: [Python for Everybody — Specialization](https://www.coursera.org/specializations/python) (University of Michigan)

- Khóa 1: "Programming for Everybody" — nếu chưa bao giờ code

### 🆓 Free:
- [Python.org Official Tutorial](https://docs.python.org/3/tutorial/) — Reference
- [Automate the Boring Stuff](https://automatetheboringstuff.com/) — Practical exercises

---

## 📅 Lịch Học Theo Ngày

### Thứ 2 — Setup & Basics (3.5 giờ)
| Thời gian | Hoạt động | Chi tiết |
|-----------|-----------|----------|
| 1.5h | 🎬 Video | Udemy Sections 1-2: Setup Python, VS Code, first script |
| 1.0h | 💻 Thực hành | Cài Python 3.12, VS Code, chạy `hello.py` |
| 0.5h | 💻 Bài tập | Viết 5 scripts: print, input, calculator |
| 0.5h | 📝 Ghi chú | Tạo file `notes/week01-basics.md` |

**Bài tập ngày:**
```python
# Bài 1: Calculator
num1 = float(input("Nhập số 1: "))
num2 = float(input("Nhập số 2: "))
op = input("Phép tính (+,-,*,/): ")
# ... implement calculator

# Bài 2: Temperature converter (C ↔ F)
# Bài 3: BMI calculator
```

### Thứ 3 — Data Structures (3.5 giờ)
| Thời gian | Hoạt động | Chi tiết |
|-----------|-----------|----------|
| 1.5h | 🎬 Video | Udemy Section 3: Strings, Lists, Dicts, Sets, Tuples |
| 1.0h | 💻 Thực hành | Exercises cho mỗi data type |
| 0.5h | 💻 Bài tập | 10 bài tập data structures |
| 0.5h | 📝 Flashcards | Tạo flashcards cho syntax |

**Bài tập ngày:**
```python
# Bài 1: Word counter - đếm từ trong một đoạn văn
text = "Python is great. Python is powerful. Python is easy."
# Output: {'Python': 3, 'is': 3, 'great.': 1, ...}

# Bài 2: List operations - filter, sort, transform
numbers = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5]
# Tìm unique, sort descending, filter > 3

# Bài 3: Dictionary manipulation
student = {"name": "Nghia", "age": 25, "scores": [85, 92, 78]}
# Tính trung bình scores, thêm field "average"
```

### Thứ 4 — Control Flow & Functions (3.5 giờ)
| Thời gian | Hoạt động | Chi tiết |
|-----------|-----------|----------|
| 1.5h | 🎬 Video | Udemy Sections 4-6: if/else, loops, functions |
| 1.0h | 💻 Thực hành | Viết 5 functions tự thiết kế |
| 0.5h | 💻 Bài tập | FizzBuzz, Fibonacci, Prime checker |
| 0.5h | 🧪 Challenge | LeetCode Easy x2 (Two Sum, Palindrome Number) |

**Bài tập ngày:**
```python
# Bài 1: FizzBuzz
def fizzbuzz(n):
    """In 1 đến n: Fizz(chia 3), Buzz(chia 5), FizzBuzz(chia 15)"""
    pass

# Bài 2: Password validator
def validate_password(password):
    """Ít nhất 8 ký tự, 1 uppercase, 1 number, 1 special char"""
    pass

# Bài 3: Fibonacci generator
def fibonacci(n):
    """Trả về n số Fibonacci đầu tiên"""
    pass
```

### Thứ 5 — OOP (3.5 giờ)
| Thời gian | Hoạt động | Chi tiết |
|-----------|-----------|----------|
| 1.5h | 🎬 Video | Udemy Section 8: Classes, Inheritance, Special Methods |
| 1.0h | 💻 Thực hành | Tạo class hierarchy: Animal → Dog, Cat |
| 0.5h | 💻 Bài tập | Bank Account class với deposit/withdraw |
| 0.5h | 📝 Diagram | Vẽ class diagram cho Todo App |

**Bài tập ngày:**
```python
# Bài 1: Bank Account
class BankAccount:
    def __init__(self, owner, balance=0):
        self.owner = owner
        self.balance = balance
    
    def deposit(self, amount): ...
    def withdraw(self, amount): ...
    def __str__(self): ...

# Bài 2: Student Management
class Student:
    def __init__(self, name, student_id): ...
    def add_grade(self, subject, grade): ...
    def get_gpa(self): ...
```

### Thứ 6 — File I/O & Modules (3.5 giờ)
| Thời gian | Hoạt động | Chi tiết |
|-----------|-----------|----------|
| 1.5h | 🎬 Video | Udemy Section 9: Modules, Packages, File I/O |
| 1.0h | 💻 Thực hành | Đọc/ghi JSON, CSV, text files |
| 0.5h | 💻 Bài tập | CSV parser + JSON config reader |
| 0.5h | 📝 Review | Ôn lại tuần, ghi chú concepts khó |

**Bài tập ngày:**
```python
# Bài 1: CSV Reader
import csv
def read_students(filepath):
    """Đọc CSV file, return list of dicts"""
    pass

# Bài 2: JSON Config
import json
def load_config(filepath="config.json"):
    """Load config, return dict, handle file not found"""
    pass

# Bài 3: Log Writer
def write_log(message, filepath="app.log"):
    """Append message với timestamp vào log file"""
    pass
```

### Thứ 7 — Milestone Project: CLI Todo App (4 giờ)
| Thời gian | Hoạt động | Chi tiết |
|-----------|-----------|----------|
| 1.0h | 📝 Design | Thiết kế features, data structure |
| 2.0h | 💻 Code | Implement toàn bộ app |
| 0.5h | 🧪 Test | Test manual tất cả features |
| 0.5h | 📤 Push | Tạo GitHub repo, push code |

**Yêu cầu Todo App:**
```
Features:

  1. Add task (title, priority: low/medium/high)
  2. List tasks (filter by status, priority)
  3. Complete task (mark as done)
  4. Delete task
  5. Save/Load from JSON file
  
Structure:
  todo_app/
  ├── main.py          # CLI interface
  ├── todo.py          # Todo class
  ├── storage.py       # JSON file operations
  └── data/
      └── todos.json   # Persistent storage
```

### Chủ Nhật — Review & Prep (2 giờ)
| Thời gian | Hoạt động |
|-----------|-----------|
| 1.0h | 🔄 Review: chạy lại tất cả bài tập, sửa lỗi |
| 0.5h | 📝 Viết weekly summary trong `notes/` |
| 0.5h | 🔮 Preview tuần 02: đọc trước về async, type hints |

---

## ✅ Đánh Giá Cuối Tuần

### Checklist Tự Kiểm Tra (phải đạt ≥ 8/10):

| # | Tiêu chí | ✅/❌ |
|---|---------|------|
| 1 | Giải thích được sự khác nhau giữa list, tuple, dict, set | |
| 2 | Viết function với default params, *args, **kwargs | |
| 3 | Tạo class với __init__, __str__, inheritance | |
| 4 | Đọc/ghi JSON file trong Python | |
| 5 | Sử dụng list comprehension | |
| 6 | Xử lý exception với try/except | |
| 7 | Import module từ file khác | |
| 8 | CLI Todo App hoạt động đầy đủ | |
| 9 | Code đã push lên GitHub | |
| 10 | Giải được ít nhất 2 bài LeetCode Easy | |

### Nếu không đạt ≥ 8/10:
- Dành thêm 2-3 ngày ôn lại phần yếu
- Xem thêm video từ "100 Days of Code" (Angela Yu) cho phần bị thiếu
- **KHÔNG skip sang tuần 02** khi chưa vững

---

## 📊 Tracking

```
Tuần 01 Progress:
├── [ ] Thứ 2: Setup & Basics
├── [ ] Thứ 3: Data Structures  
├── [ ] Thứ 4: Control Flow & Functions
├── [ ] Thứ 5: OOP
├── [ ] Thứ 6: File I/O & Modules
├── [ ] Thứ 7: 🏆 CLI Todo App
└── [ ] Chủ Nhật: Review & Assessment
```

---

> **Tiếp theo**: [Tuần 02: Python Nâng Cao →](Tuan-02-Python-Nang-Cao.md)
