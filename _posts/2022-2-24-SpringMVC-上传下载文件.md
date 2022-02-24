当我们需要实现上传下载文件的功能时：

## 上传文件：

- 上传代码将 **type** 设置为 **“file”**，为表单提价设置为 **enctype="multipart/form-data"**
- jsp 代码
    
    ```html
    <form action=" ${pageContext.request.contextPath}/book/uploadbook" enctype="multipart/form-data" method="post">
        <div class="form-group">
            <label for="bookname">书名</label>
            <input type="text" id="bookname" class="form-control" name="bookname" required="">
        </div>
        <div class="form-group">
            <label for="selectbook">选择文件</label>
            <input id="selectbook" type="file" name="bookfile">
        </div>
        <div>
            <button class="btn btn-primary btn-lg" type="submit">上传</button>
        </div>
    </form>
    ```
    
- Controller 层代码：
    
    ```java
    @RequestMapping(value = "/uploadbook", method = RequestMethod.POST)
    public String upload(@RequestParam("bookfile") MultipartFile file,
                         String bookname, HttpServletRequest request, Model model) {
        if (!file.isEmpty()) {
            String contextPath = request.getContextPath();//"/SpringMvcFileUpload"
            String servletPath = request.getServletPath();//"/gotoAction"
            String scheme = request.getScheme();//"http"
            String storePath = "/Users/chenyang/study/labweb1/web/books";//存放我们上传的文件路径
            String fileName = file.getOriginalFilename();
            String userName = null;

            if (fileName == null) {
                model.addAttribute("error", 6); // 文件名为空
                return "错误页";
            }
            File filepath = new File(storePath, fileName);
            if (!filepath.getParentFile().exists()) {

                filepath.getParentFile().mkdirs();//如果目录不存在，创建目录
                return "错误页";
            }
            try {
                file.transferTo(new File(storePath + File.separator + filenames));//把文件写入目标文件地址
            } catch (Exception e) {
                e.printStackTrace();
                return "uploadbook";
            }
            return "正确页";
        } else {
            model.addAttribute("error", 8);
            return "错误页";
        }
    }
    ```
    

## 下载文件

- jsp代码：
    
    ```html
    <td>
        <a class="btn btn-info" href="/book/${book.bookId}/download">下载</a>
    </td>
    ```
    
- Controller层代码
  ```java
      @RequestMapping(value="/{bookId}/download",method=RequestMethod.GET) //匹配的是href中的download请求
    public ResponseEntity<byte[]> download(HttpServletRequest request, @PathVariable("bookId") int bookId,
                                           Model model) throws IOException {
        String filename = Integer.toString(bookId);
        String downloadFilePath="/Users/chenyang/study/labweb1/web/books";//从我们的上传文件夹中去取

        File file = new File(downloadFilePath+File.separator+filename + ".pdf");//新建一个文件
        System.out.println(file.getAbsolutePath());
        HttpHeaders headers = new HttpHeaders();//http头信息

        String downloadFileName = new String(filename.getBytes(StandardCharsets.UTF_8), StandardCharsets.ISO_8859_1);//设置编码

        headers.setContentDispositionFormData("attachment", downloadFileName);

        headers.setContentType(MediaType.APPLICATION_OCTET_STREAM);

        //MediaType:互联网媒介类型  contentType：具体请求中的媒体类型信息

        return new ResponseEntity<byte[]>(FileUtils.readFileToByteArray(file),headers, HttpStatus.CREATED);
}
  ```
