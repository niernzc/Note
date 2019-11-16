Go通过encoding/json库支持json

### 把结构映射为json

对于名字为<name>的json键，只需要在结构里创建一个任意名字的字段，并将该字段的***结构标签***设置为‘json:“<name>”’，即可。

