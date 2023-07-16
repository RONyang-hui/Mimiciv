Sub HighlightDuplicateHeaders()
    Dim ws As Worksheet
    Dim rng As Range
    Dim cell As Range
    Dim dict As Object
    
    On Error GoTo ErrHandler '指定错误处理程序的位置
    
    Set ws = ActiveSheet '或者指定工作表名称，如Set ws = Worksheets("Sheet1")
    Set rng = ws.Range("A1", ws.Cells(1, ws.Columns.Count).End(xlToLeft)) '获取第一行的列标题范围
    Set dict = CreateObject("Scripting.Dictionary") '创建字典对象
    
    For Each cell In rng '遍历每个列标题
        If dict.exists(cell.Value) Then '如果字典中已经存在该值
            cell.Interior.Color = vbYellow '则给单元格添加黄色背景色
        Else '否则
            dict.Add cell.Value, 1 '将该值添加到字典中，值为1
        End If
    Next cell
    
    Set dict = Nothing '释放字典对象
    
    Exit Sub '正常退出子程序
    
ErrHandler: '错误处理程序开始
    MsgBox Err.Description, vbCritical, "出错了" '显示错误信息
    Resume Next '继续执行下一条语句
End Sub
