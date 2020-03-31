# PostMan使用技巧

## 添加环境变量

添加环境变量，比如请求的URL，并对应好环境，之后请求某个环境时，只要切换一下环境就OK了

## 使用Pre-request Script

作用是，postman在发出请求之前可执行此处的脚本，可用来个添加参数等  
实例:签名请求

```js
var partner_key = pm.variables.get("partner_key");

console.log(partner_key)
var fdata = pm.request.body.formdata.members

var params = {}
for (let i in fdata) {
    if (fdata[i]["key"] != "xdSign") {
        params[fdata[i]["key"]] = fdata[i]["value"]
    }
}

var signData = {}
Object.keys(params).sort().map(key => {
  signData[key]=params[key]
})

var signStr = Object.keys(signData).map(function (key) {
    return "".concat(encodeURIComponent(key), "=").concat(encodeURIComponent(signData[key]));
}).join('&');


if (signStr) {
    sign = CryptoJS.MD5(signStr + "&partner_key="+partner_key).toString()
} else {
    sign = CryptoJS.MD5("partner_key="+partner_key).toString()
}

pm.environment.set("sign", sign)

```
