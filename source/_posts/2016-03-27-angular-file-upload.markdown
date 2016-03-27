---
layout: post
title: "Angular File Upload"
date: 2016-03-27 21:03:04 +0800
comments: true
categories: angular
---

網站上有時候會用到上傳檔案功能。  
透過 angular，可以使用[angular-file-upload](https://github.com/nervgh/angular-file-upload) 很方便的就建立出來

<!-- more -->  

這個 module 也有提供很多 API 可以使用 [Module-API](https://github.com/nervgh/angular-file-upload/wiki/Module-API)

#controller
```js
app
.controller('VideoShowCtrl',
  [        '$scope', '$http', 'FileUploader', '$stateParams',
  function( $scope ,  $http ,  FileUploader ,  $stateParams){
	
	//透過 $stateParams (ui-router) 可以得到 router 上的 id
	var postID = $stateParams.id     
	var uploader = new FileUploader({
        url: '/posts/show' + postID',
        formData: [],
        headers: {
            'X-CSRF-TOKEN' : document.querySelector("meta[name=csrf-token]").content
        }
    });
	
	//內建提供上傳檔案前可以做哪些事
    uploader.onBeforeUploadItem = function(item){
    	//若是傳的是 array，接到的會變成 string，可能是放在 formData 的原因@@
        item.formData.push({ title: 'hello' })
    };
	
	//內建提供上傳檔案後做哪些事
    uploader.onAfterAddingFile = function(item){
        if(item.uploader.queue.length > 1){ item.uploader.queue.shift() }
    };

    $scope.uploader = uploader

}]);

```

#view
```js
<input type="file" nv-file-select="" uploader="uploader" />
```

參考文件：  
[URL Routing](https://github.com/angular-ui/ui-router/wiki/URL-Routing)  

gem：  
[angularjs-file-upload-rails](https://github.com/Marthyn/angularjs-file-upload-rails)  
[angular-file-upload](https://github.com/nervgh/angular-file-upload)