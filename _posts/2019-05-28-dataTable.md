---
layout: post
title: DataTables 的使用
category: 技术
tags: [JavaScript]
description: 

---







> dataTables 是一个功能比较强大的JavaScript库，用于在页面上动态的构建表格，本文总结一下使用的基本流程



#### 1、导包

##### (1) 下载

要使用dataTables必须包含Jquery和datatable的js文件 ,下载dataTables的包 https://datatables.net/download/，也可以直接从 https://github.com/wongkee/frontpage 仓库中dataTable文件夹下下载.

##### (2) 在HTML文件中引入

```html
  <link rel="stylesheet" type="text/css" href="DataTables/datatables.min.css">
  <script type="text/javascript" charset="utf8" src="JQuery/jquery-3.4.1.min.js"></script>
  <script type="text/javascript" charset="utf8" src="DataTables/datatables.min.js"></script>
```



####  2、创建容器

一般将dataTables 放置到div容器中，并且通过调整div的大小来设置表格的尺寸，表格的大小就设置为 100%即可

```html
<div class="container">
  <table id="" class="display" style="width:100%">
        <thead>
            <tr>
              <!--dataTable 也完全支持表头中使用跨行和跨列-->
                <th>Name</th>
                <th>Position</th>
                <th>Office</th>
                <th>Age</th>
                <th>Salary</th>
            </tr>
        </thead>
        <tbody>
        </tbody>
        <tfoot>
            <tr>
                <th>Name</th>
                <th>Position</th>
                <th>Office</th>
                <th>Age</th>
                <th>Salary</th>
            </tr>
        </tfoot>  
</div>

```



#### 3、添加数据

1. 如上，可以直接在表格中添加数据

2. 也可以直接定义数据用于添加

   ```
       var data = [
        [
           "Tiger Nixon",
           "System Architect",
           "Edinburgh",
           "5421",
           "2011/04/25",
           "$3,120"
        ],
        [
           "Garrett Winters",
           "Director",
           "Edinburgh",
           "8422",
           "2011/07/25",
           "$5,300"
        ],
        [
           "Thor Walton",
           "Developer",
           "Edinburgh",
           "61",
           "2011/07/25",
           "$98,540" 
        ],
        [
          "Finn Camacho",
           "Developer",
           "Edinburgh",
           "47",
           "2011/07/25",
           "$87,500" 
        ]       
      ];
   ```

   

3. 数据量过大时可以单独放在一个文件中，并且使用 Ajax获取数据



#### 4、设置属性

通过DataTable方法设置设置表中的数据以及排序，分页等功能

```javascript
$(document).ready(function() {
   // $('#table_id').DataTable({ //使用id一个设置一个表
   $('table.display').DataTable({ //使用table中的class一下设置多个表
      data:data, //如果使用javascript设置数据将会把表中原有的数据给覆盖掉
     // "ajax":'./ajax.txt',   //直接在静态页面加载会加载失败  需要服务器端使用http协议进行读取
    //  "paging":   false,  
    //  "ordering": false, 关闭排序
      "order":[[3,'asc']], //下标为3的一列以升序排列   [3，'desc'] 以降序排列  默认值[[0,'asc']] ,也可设置多列排序 [[3,'asc'],[5,'desc']]
   //   "info":     false,

       "columnDefs": [  //单独设置列的属性
       
            {
                "targets": [1],
                "visible": false,  //设置列为不可见
                "searchable": false //设置列为不可搜索
            },
            {
              "targets":[3],
               "orderData":[3,0] //默认按第三列排序，第三列相同时按照第一列进行排序
            },
            {
                "targets": [4],
                "visible": false
            }
        ],

        "dom": '<"top"i>rt<"bottom"flp><"clear">',//用于设置修饰元素的位置
        "stateSave":true,  //保存表格的状态，分页等，再次进入时仍位于该状态
        "pagingType":"full_numbers"// 分页的格式 显示 第一个前一页，后一页 最后一页 也可参考手册定制 numbers simple simple_numbers full full_numbers first_last_numbers
        //分页也可用滚轮展示
    });
} );
```

#### 参考资料

[DataTables](https://datatables.net/)