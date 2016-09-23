```javascript
/**
 * Created by sivan.liao on 16/9/12.
 */
;
'use strict';
var dduFunc = {currentPage: 1};
/***
 *  首页点击 新闻
 */
var current = 1;
var slider_length = 0;
Date.prototype.format = function(format) {
    var date = {
        "M+": this.getMonth() + 1,
        "d+": this.getDate(),
        "h+": this.getHours(),
        "m+": this.getMinutes(),
        "s+": this.getSeconds(),
        "q+": Math.floor((this.getMonth() + 3) / 3),
        "S+": this.getMilliseconds()
    };
    if (/(y+)/i.test(format)) {
        format = format.replace(RegExp.$1, (this.getFullYear() + '').substr(4 - RegExp.$1.length));
    }
    for (var k in date) {
        if (new RegExp("(" + k + ")").test(format)) {
            format = format.replace(RegExp.$1, RegExp.$1.length == 1
                ? date[k] : ("00" + date[k]).substr(("" + date[k]).length));
        }
    }
    return format;
}

dduFunc.news_slider_click = function(num, cls) {
    var total = $('.' + cls).length;
    var temp = current + num;

    current = temp;

    if(temp <= 1) {
        current = 1;
        $(".index_toutiao_right_gray").css('display', 'none');
        $(".index_toutiao_right_highlight").css('display', '');
        $(".index_toutiao_left_gray").css('display', '');
        $(".index_toutiao_left_highlight").css('display', 'none');

    } else if(temp >= total) {
        $(".index_toutiao_right_gray").css('display', '');
        $(".index_toutiao_right_highlight").css('display', 'none');
        $(".index_toutiao_left_gray").css('display', 'none');
        $(".index_toutiao_left_highlight").css('display', '');
        current = total;
    } else {
        $(".index_toutiao_left_gray").css('display', 'none');
        $(".index_toutiao_right_gray").css('display', 'none');
        $(".index_toutiao_left_highlight").css('display', '');
        $(".index_toutiao_right_highlight").css('display', '');
    }
    $.each($('.' + cls), function(i, v) {
        if(i === (current -1) ) {
            $(v).addClass('active');
        } else {
            $(v).removeClass('active');
        }
    });
};
dduFunc.tplConfig = {
    newsTpl: [
        {
            author_text: "author",
            news_link_text: 'show more',
            inputtime_text: 'datetime',
        }
    ]
};
//--------------------------------------------//
// 模板
dduFunc.defaultTemplates = {
    // 带有日期的格式化
    newsTpl: function() {
        return '<li class="border-bottom-2">' +
            '<div class="col-sm-12"> ' +
            '<div class="col-sm-8 text-left"> ' +
            '<h5>#title#</h5> ' +
            '</div> ' +
            '</div> ' +
            '<p  class="text-left ddu-font-12"> <img src="#thumb#" class="img-responsive"/> ' +
            '<span class="text-left">#author_text#:#username#|#inputtime_text#:#inputtime_format_date# ' +
            '</span> </p> <p class="text-left ddu-font-12">#description#</p> ' +
            '<p class="text-right col-sm-12 ddu-font-12 ddu-color">' +
            '<a href="#url#" title="#title#">#news_link_text#&nbsp;' +
            '<img src="statics/dduwork/source/images/Arrow-Right.png" />' +
            '</a>' +
            '</p> ' +
            '</li>';
    },
    // 带有幻灯的兼容
    memberTpl: function() {
        return '<div id="community-members-item-#k#"class="col-sm-4"> ' +
            '<div class="thumbnail" style="position:relative;padding-top:30px;padding-bottom: 60px;"> ' +
            '<img src="#image#" style="height:42px;"/> ' +
            '<div class="caption"> ' +
            '<a href="#url#">#catname#</a> ' +
            '</div> </div> </div>';

    }
};
/**
 * 格式化类库
 * @returns {*}
 */
dduFunc.getFormatConfig = function() {
    var formatConfig = {
        format_date: function (timestamp) {
            var newDate = new Date();
            newDate.setTime(timestamp * 1000);
            return newDate.format("yyyy-MM-dd hh:mm:ss");
        }
    }
    return formatConfig;
}
/**
 *
 * @param formatType
 * @returns {*}
 */
dduFunc.format = function(formatType) {
    var config = this.getFormatConfig();
    if(config[formatType] !== 'undefined' || config[formatType] !== undefined) {
        return config[formatType];
    }
    return false;
}
//操作模版
dduFunc.opTemplate = function() {
    var self = this;
    return {
        /**
         * 增加模板
         * @param templateFlag
         * @param template
         * @returns {*}
         */
        addTpl: function(templateFlag, template) {
            if(this.hasTpl(templateFlag)) {
                return templateFlag;
            }
            self.defaultTemplates[templateFlag] = template;
        },
        /**
         * 模板key
         * @param templateFlag
         * @returns {boolean}
         */
        hasTpl: function(templateFlag) {
            if(self.defaultTemplates[templateFlag] != undefined) {
                return true;
            }
            return false;
        },
        /**
         * 获取模板内容
         * @param templateFlag
         * @returns {*}
         */
        getTpl: function(templateFlag) {
            return self.defaultTemplates[templateFlag]();
        },
        /**
         * 移除模板
         * @param templateFlag
         */
        removeTpl: function(templateFlag) {
            delete self.defaultTemplates[templateFlag];
        },
        /**
         * 替换模版
         * @param data
         * @param template
         * @returns {*}
         */
        replaceTpl: function(data, template, pagesize) {
            // 当前已展示的数量
            var currentDisplayNum = null;
            if(pagesize != null) {
                var currentDisplayNum = pagesize * self.currentPage;
            }
            var tplHtml = [];
            // 获取到需要format的field
            var formatInfo = this.getFormatFields(template);

            var tplOps = this;
            $.each(data, function( i, v ) {
                var initTpl = template;
                if(currentDisplayNum != null) {
                    initTpl = initTpl.replace(/#k#/, parseInt(i + currentDisplayNum));
                }
                $.each(v, function( field, value ) {
                    var re = new RegExp("#" + field + "#");
                    var info = null;
                    info = tplOps.checkFieldIn(field, formatInfo.formatList);
                    if(info) {
                        re = new RegExp('#'+field+'_'+info.format+'#');
                        value = self.format(info.format)(value);
                    }
                    initTpl = initTpl.replace(re, value);

                });
                tplHtml.push(initTpl);
            });
            return tplHtml.join();
        },
        checkFieldIn: function(field, formatList) {
            if(formatList.length === 0) return false;
            var res = null
            $.each(formatList, function(i, v) {
                if(v.field === field) {
                    res = v;
                }
            });
            return res;
        },
        /**
         * 获取需要格式化的字段
         * @param template
         * @returns {{template: *, fields: *, formatType: *}}
         */
        getFormatFields: function(template) {
            var formatConfig = self.getFormatConfig();
            var initTpl = template, formatList = new Array();
            $.each(formatConfig, function(type, config) {
                var re = new RegExp("#([a-z]+)_(" + type + ")#");
                var match = new Array();
                match = initTpl.match(re);
                if(match != null) {
                    formatList.push({field: match[1], format: match[2]});
                }
            });
            return {
                template: template,
                formatList: formatList
            };
        }
    }
}
/**
 * 获取更多
 * @param obj 当前操作对象
 * @param appendId 追加内容的id
 * @param fetchUrl 获取数据的url
 * @param templateFlag 模板key
 * @param template 自定义展示模版
 * @param pagesize 每页展示数量
 */
dduFunc.getMoreList = function(obj, appendId, fetchUrl, templateFlag, template, pagesize) {
    var self = this;
    var op = self.opTemplate();
    if(!template && templateFlag && op.hasTpl(templateFlag)) template = op.getTpl(templateFlag);
    // 模板解析
    var templatePattern = self.tplConfig[templateFlag];
    // showloading
    $.post(fetchUrl + '&pagesize='+pagesize+'&page=' + self.currentPage, function(res) {
        if(res.status === 0 && res.data.length) {
            // config replace
            template = op.replaceTpl(templatePattern, template, null);
            var html = op.replaceTpl(res.data, template, pagesize);
            $('#' + appendId).append(html);
            self.currentPage++;
        } else {
            $(obj).html(' no more ');
        }
    });
}
;

```
### example
```html
        <h4 id="show-more" onclick="dduFunc.getMoreList(this, 'blog_list_page_ul', '{siteurl($siteid)}/api.php?op=get_more&thumb=1&&tag=content_lists&catid={$catid}', 'newsTpl', null, 3);">{L('show_more')}</h4>

```
