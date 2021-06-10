# springboot 发布到 k8s

## 开发 spring boot代码

### idea 创建项目

```
File-> new -> Projetc -> Spring Initiallzr -> next
```

```
# set project
```

<img src="images/image-20210610134037812.png" alt="image-20210610134037812" style="zoom: 50%;" />

 ![image-20210610131226471](images/image-20210610131226471.png)



### 代码

```java
package com.example.demo.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "hello spring boot";
    }
}

```

 ![image-20210610134213039](images/image-20210610134213039.png)

 ![image-20210610134246562](images/image-20210610134246562.png)





## 代码发布到k8s

