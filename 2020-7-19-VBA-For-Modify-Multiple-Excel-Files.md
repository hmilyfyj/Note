---
title: 2020-7-19-VBA-For-Modify-Multiple-Excel-Files
tags: 新建,模板,小书匠
grammar_cjkRuby: true
date: 2020-07-19 18:38
tags: Excel,Office
categories: 折腾
---

最近收到了朋友的求助，需求是把批量多个 excel 文件中的同一位置的单元格改成其他内容，新的内容中包含公式和段落格式。

<!-- more -->

---

待补充

```vba
Sub AdjustMultipleFiles()
    Dim wbLoopBook, templateWb As Workbook
    Dim wsLoopSheet As Worksheet
    Dim formulaRngFromSource, formulaRngToDest As Range
        
    With Application
       .ScreenUpdating = False: .DisplayAlerts = False: .EnableEvents = False
    End With
    
    'init params
    Path = Application.ThisWorkbook.Path
    OldPath = Path & "\old\"
    NewPath = Path & "\new\"
    TemplateFile = Path & "\模板.XLS"

    With CreateObject("Scripting.FileSystemObject")
        '// Change path to suit
        For Each File In .GetFolder(OldPath).Files
            '// ALL Excel 2007 files
            '// If .GetExtensionName(File) = "xls" Then
            If True Then
                '// Open Workbook x and Set a Workbook variable to it
                ' Open workbook to edit
                Set wbLoopBook = Workbooks.Open(Filename:=File.Path, UpdateLinks:=0)

                newFileName = .GetFileName(File)
                
                '## Open template workbooks first:
                Set templateWb = Workbooks.Open(TemplateFile)
                
                ' Copy formulas
                Set formulaRngFromSource = templateWb.Sheets("第1页").Range("B33:B35")
                Set formulaRngToDest = wbLoopBook.Sheets("第1页").Range("B33:B35")
                
                i = 1
                For Each R In formulaRngFromSource
                     formulaRngToDest(i).Formula = R.Formula
                     i = i + 1
                Next R
                
                wbLoopBook.SaveAs Filename:=NewPath & newFileName
                '// Close Workbook & Save
                wbLoopBook.Close SaveChanges:=False
                '// Release object variable
                Set wbLoopBook = Nothing
            End If
        Next File
    End With
    
    'x.Close
    'https://stackoverflow.com/questions/19351832/copy-from-one-workbook-and-paste-into-another
    templateWb.Close


    With Application
        .ScreenUpdating = True: .DisplayAlerts = True: .EnableEvents = True
    End With
    
    MsgBox ("转换完毕~")
End Sub
```