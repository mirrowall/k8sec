# API
## 代码结构

## 代码分析
initGraph()　　初始化
controller.StartGin
    -> modules/api/app/controller/routes.go
	graph.Routes(r)
	uic.Routes(r)
	template.Routes(r)
	strategy.Routes(r)
	host.Routes(r)
	expression.Routes(r)
	mockcfg.Routes(r)
	dashboard_graph.Routes(r)
	dashboard_screen.Routes(r)
	alarm.Routes(r)
