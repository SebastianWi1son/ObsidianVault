## 1. 构造函数声明与定义分离
```cpp
// --- .h ---
struct Student {
	void printInfo() const;
}
```

```cpp
// --- .cpp ---
void Student::printInfo() const {
	std::cout << "ID: " << id << ...
}
```
