# transformer

## pre vs post layer norm
几乎所有的transformer模型都使用pre layer norm，也就是在每个子层（self-attention和feedforward network）之前进行layer normalization。

### double norm

前后都进行layer normalization，虽然可以提高模型的稳定性，但会增加计算成本。

![alt text](images/image-19.png)

residual path上进行layer normalization: residiual path上的值是identity connection mapping的结果，也就是说，可以在全network layers上一贯地进行layer normalization，而不需要担心梯度消失的问题。

### layer norm vs rms norm

