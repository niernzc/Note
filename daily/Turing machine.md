题：判断一个五位的二进制数是否为奇数，图灵机。

解：

* Q={q<sub>0</sub>, q<sub>1</sub>, q<sub>2</sub>, q<sub>3</sub>, q<sub>4</sub>, q<sub>accept</sub>, q<sub>reject</sub>}

* Σ={0,1}

* Γ={0,1,}

* δ={
      q<sub>0</sub>, 0,1 $\Rightarrow$ q<sub>1</sub>, R

  ​	q<sub>1</sub>, 0,1 $\Rightarrow$ q<sub>2</sub>, R

  ​	q<sub>2</sub>, 0,1 $\Rightarrow$ q<sub>3</sub>, R	

  ​	q<sub>3</sub>, 0,1 $\Rightarrow$ q<sub>4</sub>, R

  ​	q<sub>4</sub>, 1 $\Rightarrow$ ,q<sub>accept</sub>, R

  ​	q<sub>4</sub>, 0 $\Rightarrow$ ,q<sub>reject</sub>, R

    }

  故，go程序：

  ```go
  package main
  
  import "fmt"
  
  type State int
  type dir int
  const(
  	Left dir = iota
  	Right
  )//定义纸带方向
  const(
  	q0 State = iota
  	q1
  	q2
  	q3
  	q4
  	qAccept
  	qReject
  )//定义状态集
  func stateTrans(s State,c string)(State, dir ){
  	switch s {
  	case q0:
  		return s+1,Right
  	case q1:
  		return s+1,Right
  	case q2:
  		return s+1,Right
  	case q3:
  		return s+1,Right
  	case q4:
  		if c=="1"{
  			return qAccept,Right
  		}else{
  			return qReject,Right
  		}
  	}
  	return qReject,Right
  }//定义状态转换表
  
  func Turing(input string)State{
  	i := 0
  	q := q0
  	d := Right
  	for q!=qAccept&&q!=qReject{
  		q,d = stateTrans(q, string(input[i]))
  		if d==Right{
  			i++
  		}else{
  			i--
  		}
  	}
  	return q
  }
  
  func main() {
  	states := []string{"q0","q1","q2","q3","q4","qAccept","qReject",}
  	var s string
  	_, _ = fmt.Scanln(&s)
  	fmt.Println(states[Turing(s)])
  }
  
  ```

  运行结果：

  ```shell
  01111
  qAccept
  ```

  ```shell
  01110
  qReject
  ```

  