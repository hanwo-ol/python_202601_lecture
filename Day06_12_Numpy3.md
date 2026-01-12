# NumPy 심화

## 1. 차원(Rank)의 계층적 정의

NumPy 배열은 대괄호 `[`의 중첩 깊이로 차원을 시각적으로 표현합니다. 차원이 늘어날 때마다 데이터가 어떻게 감싸지는지(Wrapping) 확인하는 예제입니다.

```python
import numpy as np

# 1. 스칼라 (Scalar, 0D Tensor)
# 방향성이 없는 단일 물리량
scalar = np.array(10)

# 2. 벡터 (Vector, 1D Tensor)
# 크기와 방향을 가짐. 1차원 축(Axis 0) 존재
vector = np.array([1, 2, 3])

# 3. 행렬 (Matrix, 2D Tensor)
# 행(Axis 0)과 열(Axis 1)로 구성된 2차원 평면 데이터
matrix = np.array([
    [1, 2, 3],  # Row 0
    [4, 5, 6]   # Row 1
])

# 4. 3차원 텐서 (3D Tensor)
# 여러 개의 행렬이 깊이(Depth) 축으로 쌓인 형태
# 구조: (Depth, Row, Column)
tensor_3d = np.array([
    [[1, 2, 3], [4, 5, 6]],    # Depth 0 (Matrix A)
    [[7, 8, 9], [10, 11, 12]]  # Depth 1 (Matrix B)
])

print(f"Scalar Shape: {scalar.shape}, Dimension: {scalar.ndim}")
print(f"Vector Shape: {vector.shape}, Dimension: {vector.ndim}")
print(f"Matrix Shape: {matrix.shape}, Dimension: {matrix.ndim}")
print(f"Tensor Shape: {tensor_3d.shape}, Dimension: {tensor_3d.ndim}")
```

## 2. 스트라이드(Strides): 메모리 접근 구조

**심화 개념:** `ndarray`는 물리적인 메모리 상에는 1렬로 저장됩니다. `shape`은 논리적인 뷰(View)일 뿐이며, 실제로 다음 인덱스로 넘어가기 위해 메모리 주소를 얼마나 이동해야 하는지는 `strides` 속성이 결정합니다.

```python
import numpy as np

# int32 타입 (하나당 4바이트)의 (3, 4) 행렬 생성
# 메모리 상: 0, 1, 2, 3, 4, ... 11 순서로 연속 저장됨
arr = np.arange(12, dtype=np.int32).reshape(3, 4)

print("배열 형태:", arr)
print(f"Shape: {arr.shape}")
print(f"Item size (bytes): {arr.itemsize}") # 요소 하나당 4바이트

# Strides 출력
# (행을 이동하는 바이트 수, 열을 이동하는 바이트 수)
print(f"Strides: {arr.strides}") 

# [해석]
# Strides가 (16, 4)인 경우:
# 1. 다음 열(Column)로 이동하려면 4바이트(요소 1개 크기) 이동
# 2. 다음 행(Row)으로 이동하려면 16바이트(4바이트 x 4열) 이동
# 즉, 데이터가 행 우선(Row-Major) 방식으로 저장되었음을 의미함.
```

## 3. 축 교환(Transpose & Swapaxes)과 구조 변경

이미지 데이터 처리(딥러닝) 등에서 빈번하게 발생하는 차원 순서 변경의 원리를 이해합니다. 데이터 값은 그대로 두고, 축의 순서만 바꿉니다.

```python
import numpy as np

# 가상 이미지 데이터 생성
# (높이 Height, 너비 Width, 채널 Channel) = (2, 2, 3)
# 예: 2x2 픽셀의 RGB 이미지
image_data = np.arange(12).reshape(2, 2, 3)

print("원본 shape:", image_data.shape) # (2, 2, 3)

# 1. Transpose (전치)
# 모든 축의 순서를 역순으로 뒤집음 (2, 2, 3) -> (3, 2, 2)
# 주로 수학적 행렬 연산 시 사용
transposed = image_data.T
print("Transpose 후 shape:", transposed.shape)

# 2. Swapaxes (축 교환)
# 특정 두 축의 위치만 맞바꿈
# 딥러닝 라이브러리(PyTorch 등)는 (Channel, Height, Width) 순서를 선호함
# 따라서 (H, W, C) -> (C, H, W)로 변환 필요
moved_axis = np.moveaxis(image_data, -1, 0) # 마지막 축(-1)을 0번째로 이동
print("Channel 우선 변환 shape:", moved_axis.shape) # (3, 2, 2)
```

## 4. 브로드캐스팅의 기하학적 이해

서로 다른 차원의 배열이 연산될 때, 차원이 확장(Stretch)되는 과정을 시뮬레이션합니다.

```python
import numpy as np

# (3, 1) 행렬: 세로로 긴 벡터
col_vector = np.array([[1], [2], [3]]) 

# (1, 3) 행렬: 가로로 긴 벡터
row_vector = np.array([[10, 20, 30]])

# 브로드캐스팅 덧셈
# 1. col_vector는 가로로 복제되어 (3, 3)이 됨
#    [[1, 1, 1],
#     [2, 2, 2],
#     [3, 3, 3]]
#
# 2. row_vector는 세로로 복제되어 (3, 3)이 됨
#    [[10, 20, 30],
#     [10, 20, 30],
#     [10, 20, 30]]
#
# 3. 두 결과가 더해짐
result = col_vector + row_vector

print(f"Col shape: {col_vector.shape}")
print(f"Row shape: {row_vector.shape}")
print(f"Result shape: {result.shape}")
print("결과 값:\n", result)
```

## 5. 4차원 텐서 (딥러닝 배치 데이터 구조)

가장 복잡한 구조인 4차원 배열의 인덱싱 예제입니다. 주로 `(Batch_Size, Channel, Height, Width)` 또는 `(Batch_Size, Height, Width, Channel)` 형태를 가집니다.

```python
import numpy as np

# 시나리오: 10장의 컬러 이미지(3채널), 크기는 28x28
# 구조: (이미지 개수, 높이, 너비, 채널)
batch_size = 10
height = 28
width = 28
channels = 3 # R, G, B

# 랜덤 데이터 생성
images = np.random.rand(batch_size, height, width, channels)
print(f"4D Tensor Shape: {images.shape}")

# 데이터 접근 연습

# 1. 첫 번째 이미지 전체 가져오기
first_image = images[0] 
print(f"첫 번째 이미지 shape: {first_image.shape}") # (28, 28, 3)

# 2. 5번째 이미지의 R(Red) 채널(0번 채널)만 가져오기
fifth_image_red = images[4, :, :, 0]
print(f"5번째 이미지 Red 채널 shape: {fifth_image_red.shape}") # (28, 28)

# 3. 모든 이미지의 정중앙 픽셀 값 가져오기
# 높이 14, 너비 14 지점
center_pixels = images[:, 14, 14, :]
print(f"모든 이미지의 중앙 픽셀 shape: {center_pixels.shape}") # (10, 3)
# 결과 해석: 10장 이미지 각각에 대해 3개 채널의 색상값이 추출됨
```
