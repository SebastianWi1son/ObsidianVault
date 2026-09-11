## 1. 基础basic
-  **'point to'** is equal to **'store'**
-  指针**大小size固定**(**默认为8 bytes 64 bits**)
-  declaration: **指向的类型**+ ****ptr**

## 2. 空指针nullptr
```cpp
int* ptr = nullptr;
ptr = ptr_int;
// *ptr X
```
-  空指针:     **可 再赋值**
-  无效指针: **不可 [[../cpp_redraft/C6 - pointer#3. 解引用dereference|解引用]]**

## 3. 解引用dereference
```cpp
int *ptr = &var_01;
int var_02 = *ptr; // dereference
// var_02 == var_01
```
-  **只有指针可以**解引用 (只是肚里有地址的变量X)
-  ****ptr**:指针**解应用的结果**是一个**对象**

## 4. 万能指针generic pointer
-  接受**任何类型**的**指针赋值**
-  [[../cpp_redraft/C6 - pointer#3. 解引用dereference|解应用]]之前必须类型转换
- ```cpp
  *((double*)generic_ptr)
  ```

## 5. pointer to Array
```cpp
int array[5] = { 1, 2, 3, 4, 5 };
int *ptr_array = array;
```
-  **ptr += 1:** 偏移一个元素，一个元素4 bytes， addr + 4 (bytes)