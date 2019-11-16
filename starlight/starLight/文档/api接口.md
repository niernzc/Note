```c
1： 启动任务及容器
/containers    -POST
Body:

{
    "request_id": {
      "type": "string",
      "description": "创建“运行容器”任务的外部请求id"
    },
### 需要添加相应的 功能
    "tags": {
      	"type": "object",
      	"description": "容器tag",
      	"properties": {
            "type": {
                "type": "string",
                "description": "3种type，code_cell，algorithm，bash_cell"
            }
        },
    },
## 可以添加相关功能，label-type
    "task_desc": {
        "type": "object",
        "description": "任务信息",
        "properties": {
            "image": {
                "type": "object",
                "description": "任务所需镜像相关信息",
                "properties": {
                    "file_path": {
                        "type": "string",
                        "description": "镜像tar包路径"
                    },
                    "name": {
                        "type": "string",
                        "description": "镜像默认名称"
                    }, 
                    "tag": {
                        "type": "string",
                        "description": "镜像默认tag"
                    },
                    "auth": {
                        "type": "string",
                        "description": "镜像作者"
                    }
                },
            },
## 目前提供了简单的镜像参数 （字符串 name:tag ）, 建议使用星光镜像，或采用默认替换的方式？
### 讨论1： 镜像的使用方式需要进行讨论？
            "task_id": {
                "type": "string",
                "description": "任务id"
            },
## 目前由接口返回，在命名方式不冲突的前提下，理论上也可由外部提供
            "cmd": {
                "type": "string",
                "description": "任务cmd"
            },
  ## 可添加
            "cpu_limit": {
                "type": "string",
                "description": "cpu限制"
            },
## 已有
            "mem_limit": {
                "type": "string",
                "description": "内存限制"
            },
## 已有
            "repo_path": {
                "type": "string",
                "description": ""
            },
## ？
            "env": {
                "type": "string",
                "description": "容器环境变量"
 ## type =》 []string
            },
## 可以添加
        "inputs": {
          "type": "array",
          "description": "输入文件信息",
          "minItems": 0,
          "items": [
            {
              "type": "object",
              "description": "文件数组",
              "properties": {
                "from_path": {
                  "type": "string",
                  "description": "输入文件名称"
                },
                "bucket": {
                  "type": "string",
                  "description": "输入文件源地址"
                }
              },
            }
          ]
        },
## 需要添加相应的功能；有类似逻辑
        "outputs": {
          "type": "array",
          "description": "输出文件信息",
          "minItems": 0,
          "items": [
            {
              "type": "object",
              "description": "文件数组",
              "properties": {
                "to_path": {
                  "type": "string",
                  "description": "输出文件目标地址"
                },
                "from_path": {
                  "type": "string",
                  "description": "输出文件源地址"
                }
              },
            }
          ]
        },
## 需要添加相应的功能；有类似逻辑
### 讨论2： 对网络存储的支持程度
    
        "download_path": {
          "type": "string",
          "description": "容器内下载目录"
        },
## 对应 workdir
        "upload_path": {
          "type": "string",
          "description": "容器内上传目录"
        }
## 对应 workdir
### 讨论3： 工作目录是否允许继承


      },
    }
}

result

{
    "timestamp": {
      "type": "string",
      "description": "响应时间"
    },
    "status": {
      "type": "number",
      "description": "返回码 200接口调用成功,400表示错误的容器类型，500接口错误",
      "enum": [
        200,
        400,
        500
      ]
    },
    "message": {
      "type": "string",
      "description": "返回信息 接口成功信息或接口错误信息"
    },
    "data": {
      "type": "object",
      "properties": {
        "container_id": {
          "type": "string",
          "description": "容器id"
        },
        "container_name": {
          "type": "string",
          "description": "容器名"
        },
        "log_path": {
          "type": "string",
          "description": "容器日志路径"
        },
### 需要调查docker k8s log 默认日志存储行为
      },
    }
}





2 .停止任务并删除容器
/containers/:container_id		-DELETE

参数：container_id 容器id

Body:

{
  "request_id": {
      "type": "string",
      "description": "接口的外部请求id"
  },
}

result

{
    "timestamp": {
      "type": "string",
      "description": "响应时间"
    },
    "status": {
      "type": "number",
      "description": "响应状态,200接口正常响应，404表示未找到相应容器，500表示错误响应"
    },
    "message": {
      "type": "string",
      "description": "响应信息"
    },
    "data": {
      "type": "object",
      "properties": {
        "container_id": {
          "type": "string",
          "description": "容器id"
        }
      },
}
}


3.获取所有容器信息 
/containers	-GET

result

{
"data": {
      "type": "object",
      "properties": {
        "containers": {
          "type": "array",
          "minItems": 1,
          "items": [
            {
              "type": "object",
              "properties": {
                "container_id": {
                  "type": "string",
  "description": "容器id"
            },
                "cpu": {
                  "type": "number",
              "description": "cpu使用情况"
                },
                "mem": {
                  "type": "string",
              "description": "mem使用情况"
                },
                "status": {
                  "type": "string",
                  "description": "容器当前状态"
                },
                "user_info": {
              	  "type": "object",
                  "description": "用户信息",
                  "properties": {
                    "uid": {
                      "type": "string",
                      "description": ""
                    }
                  }  
                },
             }
        },
    },
    "message": {
      "type": "string",
  "description": "返回信息 接口成功信息或接口错误信息"
    },
    "status": {
      "type": "number",
  "description": "响应状态,200接口正常响应，500表示错误响应"
    },
    "timestamp": {
      "type": "string",
  "description": "响应时间"
    }
}


4.获取指定容器信息 

/containers/:container_id		-GET
参数： container_id : 容器id

result

{
    "timestamp": {
      "type": "string",
      "description": "响应时间"
    },
    "status": {
      "type": "number",
      "description": "返回码 200获取信息成功,404表示未找到相应信息，500表示接口错误"
    },
    "message": {
      "type": "string",
      "description": "响应信息"
    },
    "data": {
      "type": "object",
      "properties": {
        "container": {
          "type": "object",
          "description": "容器信息",
          "properties": {
            "user_info": {
              "type": "object",
              "description": "用户信息",
              "properties": {
                "uid": {
                  "type": "string",
                  "description": ""
                }
              }
            },
            "status": {
              "type": "string",
              "description": "当前容器状态"
            },
            "mem": {
              "type": "string",
              "description": "mem使用情况"
            },
            "cpu": {
              "type": "number",
              "description": "cpu使用情况"
            },
            "container_id": {
              "type": "string",
              "description": "容器id"
            }
          }
        }
      },
    }
}


5.获取指定容器日志 
/logs		-GET

请求参数：
container_id : 容器id
start_time ：所需日志开始时间
end_time ：所需日志结束时间，若为空则代表以请求接口时间为准
size ：所需日志条数，若为空则返回时间段内所有日志
result
{
    "timestamp": {
      "type": "string",
      "description": "响应时间"
    },
    "status": {
      "type": "number",
      "description": "返回码 200获取信息成功,404表示未找到相应容器信息，500表示接口错误"
    },
    "message": {
      "type": "string",
      "description": "响应信息"
    },
    "data": {
      "type": "object",
      "properties": {
        "logs": {
          "type": "array",
          "description": "日志列表",
          "minItems": 0,
		  "items": [
 		  	{
"type": "object",
    "properties": {
    	"log_time": {
        	"type": "string",
  		"description": "日志记录时间戳"
            	},
                	"log_type": {
                  		"type": "string",
              		"description": "日志类型（info,error,warning等）"
                	},
"log_message": {
    	"type": "string",
            "description": "完整日志信息"
            },
		}
}
 ]
}
   }
},
}





	

```

