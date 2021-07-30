# HTML

## 替换表格

EXCEL另存为生成表格

替换不需要的tag

```html
<td [^<]*>
->
<td>

<table width="100%" border="1" cellpadding="0" cellspacing="0" style="font-size:12px">
<tr Align="center">
	<td><b>SDK名称</b></td>
	<td><b>所属公司</b></td>
	<td><b>获取信息</b></td>
	<td><b>服务类型</b></td>
	<td><b>隐私政策地址</b></td>
</tr>
```

