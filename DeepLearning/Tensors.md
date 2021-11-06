# Tensors

### Tensors

- **Tensors are a specialized data structure that are very similar to arrays and matrices.**
- **Tensors are similar to NumPy’s ndarrays, except that tensors can run on GPUs or other specialized hardware to accelerate computing**

```python
import torch
import numpy as np
```

### Tensor initialization

- Directly from data

```python

data = [[1, 2], [3, 4]]
x_data = torch.tensor(data)
```

- From a numpy array

```python
np_array = np.array(data)
x_np = torch.from_numpy(np_array)
```

- From another tensor

```python
x_ones = torch.ones_like(x_data) # retains the properties of x_data
print(f"Ones Tensor: \n {x_ones} \n")

*x_rand = torch.rand_like(x_data, dtype=torch.float) # overrides the datatype of x_data
print(f"Random Tensor: \n {x_rand} \n")*

#
#Ones Tensor:
# tensor([[1, 1],
#        [1, 1]])
#
#Random Tensor:
# tensor([[0.2747, 0.1986],
#        [0.9290, 0.1171]])
```

- With random or constant values

```python
shape = (2, 3,)
rand_tensors = torch.rand(shape)
ones_tensors = torch.ones(shape)
zeroes_tensors = torch.zeros(shape)

#Random Tensor:
 #tensor([[0.2701, 0.0116, 0.7739],
 #       [0.9594, 0.2504, 0.1275]])

#Ones Tensor:
 #tensor([[1., 1., 1.],
 #       [1., 1., 1.]])

#Zeros Tensor:
# tensor([[0., 0., 0.],
 #       [0., 0., 0.]])
```

### Tensor Attributes

- **Tensor attributes describe their shape, datatype, and the device on which they are stored.**

```python
tensor = torch.rand(3, 4)

print(f"Shape of tensor: {tensor.shape}")
print(f"Datatype of tensor: {tensor.dtype}")
print(f"Device tensor is stored on: {tensor.device}")

#Shape of tensor: torch.Size([3, 4])
#Datatype of tensor: torch.float32
#Device tensor is stored on: cpu
```

### Tensor Operations

- **Over 100 tensor operations, including transposing, indexing, slicing, mathematical operations, linear algebra, random sampling, and more are comprehensively described [here](https://pytorch.org/docs/stable/torch.html).**

```python
# We move our tensor to the GPU if available
if torch.cuda.is_available():
  tensor = tensor.to('cuda')
  print(f"Device tensor is stored on: {tensor.device}")
```

- **Standard numpy-like indexing and slicing**

```python
tensor = torch.ones(4, 4)
tensor[:, 1] = 0
print(tensor)

#tensor([[1., 0., 1., 1.],
#        [1., 0., 1., 1.],
#        [1., 0., 1., 1.],
#       [1., 0., 1., 1.]])
```

- **Joining tensors**

```python
t1 = torch.cat([tensor, tensor, tensor], dim = 1)
print(t1)

#tensor([[1., 0., 1., 1., 1., 0., 1., 1., 1., 0., 1., 1.],
 #       [1., 0., 1., 1., 1., 0., 1., 1., 1., 0., 1., 1.],
 #       [1., 0., 1., 1., 1., 0., 1., 1., 1., 0., 1., 1.],
 #       [1., 0., 1., 1., 1., 0., 1., 1., 1., 0., 1., 1.]])
```

- **Multiplying tensors**

```python
# This computes the element-wise product
print(f"tensor.mul(tensor) \n {tensor.mul(tensor)} \n")
# Alternative syntax:
print(f"tensor * tensor \n {tensor * tensor}")

# the matrix multiplication between two tensors
print(f"tensor.matmul(tensor.T) \n {tensor.matmul(tensor.T)} \n")
# Alternative syntax:
print(f"tensor @ tensor.T \n {tensor @ tensor.T}")
```

- **In-place operations**

```python
#Operations that have a _ suffix are in-place. 
#For example: x.copy_(y), x.t_(), will change x.

tensor.add_(5);
```

### Bridge with Numpy

- Tensor to Numpy array

```python
t = torch.ones(5)
n = t.numpy()

#t: tensor([1., 1., 1., 1., 1.])
#n: [1. 1. 1. 1. 1.]
```

- **A change in the tensor reflects in the NumPy array.**

```python
t.add_(1)

#t: tensor([2., 2., 2., 2., 2.])
#n: [2. 2. 2. 2. 2.]
```

- Numpy to Tensor

```python
n = np.ones(5)
t = torch.from_numpy(n)

np.add(n, 1, out = n)

#t: tensor([2., 2., 2., 2., 2.], dtype=torch.float64)
#n: [2. 2. 2. 2. 2.]
```

- **Changes in the NumPy array reflects in the tensor.**