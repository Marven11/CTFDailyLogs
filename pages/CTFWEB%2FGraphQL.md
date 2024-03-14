# 内省
	- 可以用于查询支持的模式
	- ```graphql
	  {__schema{queryType{name}}}
	  ```
	- ```graphql
	  query {
	    __schema {
	      queryType {
	        name
	        kind
	        fields {
	          name
	        }
	      }
	      mutationType {
	        name
	        kind
	        fields {
	          name
	        }
	      }
	      subscriptionType {
	        name
	        kind
	        fields {
	          name
	        }
	      }
	    }
	  }
	  ```
	- 可以用[这个工具](https://github.com/graphql-kit/graphql-voyager)可视化
- # 资源
	- https://tari.moe/p/2022/graphql-dvga.html
- # 参考
	- https://xz.aliyun.com/t/13733
	- https://xz.aliyun.com/t/13706