---
title: HTML5表单验证反馈
date: 2018-08-14 21:53:03
tags:
- '学习'
- '整理'
categories:
- 'HTML5'
---
### validity对象
通过validity对象，通过下面的valid可以查看验证是否通过，如果八种验证都通过返回true，有一种失败则返回false
* oText.addEventListener('invalid', fn, false);
* ev.preventDefault()
* valueMissing: 输入值为空时
* typeMismatch: 控件值与预期类型不匹配
* patterMismatch: 输入值不满足pattern正则
* tooLong: 超过maxLength最大限制
* rangeUnderflow: 验证的range最小值
* rangeOverflow: 验证的range最大值
* setMismatch: 验证range的当前值是否符合min、max、step的规则
* customError: 不符合自定义验证--setCustomValidity()设置自定义验证

{% codeblock lang:html %}
<form action="">
    <input type="text" name="" id="text" required />
    <input type="submit" value="提交" />
</form>

<script>
    let oText = document.getElementById('text');
    oText.addEventListener('invalid', fn, false);

    function fn() {
        console.log(this.validity);
        console.log(this.validity.valid);
    }
</script>
{% endcodeblock %}