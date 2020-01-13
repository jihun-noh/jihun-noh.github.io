---
layout: post
title: javascript post 요청
category: Python
tags: [pyhthon, django, javascript]
---
Form 방식  
function sendForm(url, data) {  
var form = document.createElement('form');  
form.setAttribute('method', 'post');  
form.setAttribute('action', url);  
document.charset = "utf-8";  
var hiddenField = document.createElement('input');  
hiddenField.setAttribute('type', 'hidden');  
hiddenField.setAttribute('name', 'marker');  
hiddenField.setAttribute('value', data);  
form.appendChild(hiddenField);  
var hiddenField1 = document.getElementsByName('csrfmiddlewaretoken');  
form.appendChild(hiddenField1[0]);  
document.body.appendChild(form);  
form.submit();  
}  
  
ajax  
function sendPost(url, data) {  
          $.ajax({  
                url:url,  
                type:'post',  
                data:{'data':data, 'csrfmiddlewaretoken': '{{ csrf_token }}'},  
                dataType: "json",  
                success: function(data) {  
                },  
                error: function(err) {  
                    alert("error!!")  
                }  
            });  
      }  
