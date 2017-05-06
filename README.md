# blog
早就想写个人博客了,只怪自己懒😭,以此勉励自己⛽️

### js
```javascript
var aasdads = function(){
    window.alert(123);
}
var stopSearchAnim = function(callback){
    setTimeout(function(){
        isSearchAnim = false;
        callback && callback();
    }, searchAnimDuration);
};

$('#nav-search-btn').on('click', function(){
    if (isSearchAnim) return;

    startSearchAnim();
    $searchWrap.addClass('on');
    stopSearchAnim(function(){
        $('.search-form-input').focus();
    });
});
```
### html
```html
<div class="asdad">asdasd</div>
```
```shell
$ npm install
```
`npm install`

* asdasd
* sddff

--------

# ff

> This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet,
> consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus.
> Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.
> 
> Donec sit amet nisl. Aliquam semper ipsum sit amet velit. Suspendisse
> id sem consectetuer libero luctus adipiscing.