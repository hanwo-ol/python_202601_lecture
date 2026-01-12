# NumPy 실습: 차원(Dimension)과 축(Axis)의 이해

## 1. 차원별 배열 생성 및 구조 확인

스칼라(0차원)부터 3차원 텐서까지 생성하여 `ndim`(차원 수)과 `shape`(형태)의 차이를 비교합니다.

```python
import numpy as np

# 1. 0차원 배열 (Scalar)
# 값이 하나뿐이며, 축(Axis)이 존재하지 않음
arr_0d = np.array(42)

# 2. 1차원 배열 (Vector)
# 하나의 축(Axis 0)만 존재함
arr_1d = np.array([1, 2, 3])

# 3. 2차원 배열 (Matrix)
# 두 개의 축(Axis 0: 행, Axis 1: 열) 존재
arr_2d = np.array([
    [1, 2, 3],
    [4, 5, 6]
])

# 4. 3차원 배열 (Tensor)
# 세 개의 축(Axis 0: 깊이/면, Axis 1: 행, Axis 2: 열) 존재
arr_3d = np.array([
    [[1, 2], [3, 4]],
    [[5, 6], [7, 8]]
])

# 구조 출력 함수
def print_info(name, arr):
    print(f"[{name}]")
    print(f" - ndim (차원 수): {arr.ndim}")
    print(f" - shape (형태): {arr.shape}")
    print("-" * 30)

print_info("0D Scalar", arr_0d)
print_info("1D Vector", arr_1d)
print_info("2D Matrix", arr_2d)
print_info("3D Tensor", arr_3d)

# [실행 결과 예상]
# [0D Scalar] -> ndim: 0, shape: ()
# [1D Vector] -> ndim: 1, shape: (3,)
# [2D Matrix] -> ndim: 2, shape: (2, 3)
# [3D Tensor] -> ndim: 3, shape: (2, 2, 2)
```

## 2. 2차원 배열에서의 축(Axis) 연산

축(Axis)을 지정하여 연산(`sum`)을 수행할 때, 데이터가 어떻게 압축(Collapse)되는지 확인합니다.

*   **`axis=0`:** 행 방향(수직)으로 연산 수행 -> 행이 사라지고 열만 남음.
*   **`axis=1`:** 열 방향(수평)으로 연산 수행 -> 열이 사라지고 행만 남음.

```python
import numpy as np

# 3행 4열의 2차원 배열 생성 (0 ~ 11)
matrix = np.arange(12).reshape(3, 4)

print("원본 배열 (shape: (3, 4)):\n", matrix)
# [[ 0  1  2  3]
#  [ 4  5  6  7]
#  [ 8  9 10 11]]

print("\n--- 축(Axis)별 합계 연산 ---")

# Case 1: axis=None (기본값) -> 모든 요소의 합
total_sum = matrix.sum()
print(f"1. 전체 합 (axis=None): {total_sum}")

# Case 2: axis=0 (수직 방향 합계)
# (3, 4) -> (4,) 형태로 변경됨 (3개의 행이 1개로 압축됨)
# 계산: [0+4+8, 1+5+9, 2+6+10, 3+7+11]
sum_axis_0 = matrix.sum(axis=0)
print(f"2. axis=0 합 (행 압축): {sum_axis_0}")
print(f"   결과 shape: {sum_axis_0.shape}")

# Case 3: axis=1 (수평 방향 합계)
# (3, 4) -> (3,) 형태로 변경됨 (4개의 열이 1개로 압축됨)
# 계산: [0+1+2+3, 4+5+6+7, 8+9+10+11]
sum_axis_1 = matrix.sum(axis=1)
print(f"3. axis=1 합 (열 압축): {sum_axis_1}")
print(f"   결과 shape: {sum_axis_1.shape}")
```

## 3. 3차원 배열과 축의 확장

3차원 데이터에서의 축의 의미를 파악합니다.

*   `shape`가 `(2, 3, 4)`인 경우:
    *   Axis 0: 크기 2 (깊이)
    *   Axis 1: 크기 3 (행)
    *   Axis 2: 크기 4 (열)

```python
import numpy as np

# 2(깊이) x 3(행) x 4(열) 형태의 3차원 배열 생성
tensor = np.arange(24).reshape(2, 3, 4)

print(f"원본 3D 배열 shape: {tensor.shape}")

# axis=0 연산: 깊이를 압축
# 두 개의 (3, 4) 면을 더하여 하나의 (3, 4) 면으로 만듦
sum_axis_0 = tensor.sum(axis=0)
print(f"axis=0 연산 결과 shape: {sum_axis_0.shape}") # (3, 4)

# axis=1 연산: 행을 압축
# 각 면 내부에서 세로 방향으로 더함
# (2, 3, 4) -> (2, 4)
sum_axis_1 = tensor.sum(axis=1)
print(f"axis=1 연산 결과 shape: {sum_axis_1.shape}") # (2, 4)

# axis=2 연산: 열을 압축
# 각 면 내부에서 가로 방향으로 더함
# (2, 3, 4) -> (2, 3)
sum_axis_2 = tensor.sum(axis=2)
print(f"axis=2 연산 결과 shape: {sum_axis_2.shape}") # (2, 3)
```

## 4. Reshape과 차원 조작 (차원의 붕괴와 재구성)

`-1`을 활용한 자동 차원 계산과 차원 평탄화 예제입니다.

```python
import numpy as np

# 데이터 준비 (0 ~ 11)
data = np.arange(12) 
print(f"초기 데이터 shape: {data.shape}") # (12,)

# 1. 차원 늘리기 (Reshape)
# -1은 '나머지 차원의 크기에 맞춰 자동 계산'을 의미
reshaped_1 = data.reshape(3, -1) # 3행 n열 -> 12/3 = 4열 자동 할당
print(f"reshape(3, -1) 결과 shape: {reshaped_1.shape}") # (3, 4)

reshaped_2 = data.reshape(-1, 2) # n행 2열 -> 12/2 = 6행 자동 할당
print(f"reshape(-1, 2) 결과 shape: {reshaped_2.shape}") # (6, 2)

reshaped_3d = data.reshape(2, 2, -1) # 2면 2행 n열 -> 12/(2*2) = 3열
print(f"reshape(2, 2, -1) 결과 shape: {reshaped_3d.shape}") # (2, 2, 3)

# 2. 차원 줄이기 (Flattening)
# 다차원 배열을 1차원으로 펼침
flattened = reshaped_3d.flatten()
print(f"flatten() 결과 shape: {flattened.shape}") # (12,)
print(f"데이터 확인: {flattened}")
```
