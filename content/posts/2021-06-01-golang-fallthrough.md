---
title: golang fallthrough用法
date: 2021-06-01
---

fallthrough：Go里面switch默认相当于每个case最后带有break，匹配成功后不会自动向下执行其他case，而是跳出整个switch, 但是可以使用fallthrough强制执行后面的case代码。

示例：切割字符串输出格式

```go
func main() {
 const text = `Galaksinin Batı Sarmal Kolu'nun bir ucunda, haritası bile çıkarılmamış ücra bir köşede, gözlerden uzak, küçük ve sarı bir güneş vardır.

Bu güneşin yörüngesinde, kabaca yüz kırksekiz milyon kilometre uzağında, tamamıyla önemsiz ve mavi-yeşil renkli, küçük bir gezegen döner.

Gezegenin maymun soyundan gelen canlıları öyle ilkeldir ki dijital kol saatinin hâlâ çok etkileyici bir buluş olduğunu düşünürler.`

 const maxWidth = 40

 var lw int // line width

 for _, r := range text {
  fmt.Printf("%c", r)

  switch lw++; {
  case lw > maxWidth && r != '\n' && unicode.IsSpace(r):
   fmt.Println()
   fallthrough
  case r == '\n':
   lw = 0
  }
 }
 fmt.Println()
}
```
