title:  Jquery的DataTable插件 AJAX 服务器分页
date: 2016-3-18 15:19:56
tags: Jquery
category: dev
---

## HTML
---
```html
<table id="test_table" class="table table-striped table-bordered table-hover" >
    <thead>
        <tr>
            <th>1</th>
            <th>2</th>
            <th>3</th>
            <th>4</th>
        </tr>
    </thead>
    <tbody>
    </tbody>
    <tfoot>
    </tfoot>
</table>
```

## JavaScript
---

```javascript
$(document).ready(function() {
        dataTable1 = $('#test_table').dataTable({
            "bPaginate": true,
            "tableTools": {
                "sSwfPath": "js/plugins/dataTables/swf/copy_csv_xls.swf"
            },
            "bProcessing": true,
            "bServerSide": true,
            "sAjaxSource": "/cgi-bin/pypy.py",
            "sPaginationType": 'full_numbers',
            "iDisplayLength" : 10,
            "bFilter" : false
});
```

## python
---
```python
json_data = {
    "aaData":dict_data,
    "iTotalRecords":iTotalRecords,
    "iTotalDisplayRecords":iTotalDisplayRecords
}
```

## 其他参考
http://sgyyz.blog.51cto.com/5069360/1408251