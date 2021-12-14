---
title: "About"
date: 2021-12-13T13:51:40+08:00
draft: false
---

# 关于我
这里是 kevin 的 blog，AKA hzbskak. 

未来方向：go后端开发
``` golang
type PersonalInformation struct {
	Name        string
	Age         int64
	Address     string
	Email       string
	Description string
}
type Skills struct {
	Skill []string
}
func Me() (PersonalInformation, Skills) {
	information := PersonalInformation{
		Name:        "Kevin",
		Age:         26,
		Address:     "BeiJing",
		Email:       "hzbskak@gmail.com",
		Description: "Like you like liking coding",
	}
	skill := Skills{
		Skill: []string{"Golang", "php", "docker", "ubuntu"},
	}
	return information, skill
}

```

致敬我的开始！

```golang
func main()  {
	fmt.Println("Hello, World!")
	return
}

```
