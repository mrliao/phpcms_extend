```javascript
'use strict';
/**
 * 	1、向上滚动，滚动条位置  监听鼠标滚动
 *  2、向下滚动，滚动条设置  监听鼠标滚动的
 *  3、圆点点滚动样式设置
 * @type {number}
 */
// 当前的条数
var curNum = 0, scrollTop = 0;
// 动画是否完成
var isFinish = true;
// 每一个滚动的高度
var perHeight = $(document.body).outerHeight(true);
//  宽度
var perWidth = $(window).width();
// 最大值
var maxNum = 0;

$(function() {
	var debugCss = false;
	// 最大的滚动条数
	maxNum = $(".btn-after-footer li").length;
	// 方便调试
	if(debugCss) {
		$('.navbar-fixed-top').removeClass('navbar-custom').addClass('navbar-default');
	}

	$.each($('.main'), function(i, v) {

		$(v).css({height:perHeight+ 'px', width: perWidth + 'px', overflow:"hidden"});

	});
	// 最大的滚动高度
	var maxHeight = perHeight * maxNum;

	// 滚动动画设置
	/**
	 * 设置滚动
	 * @param direct
	 * @param distance
	 * @param times
	 */
	var scroll = function(direct, distance, times) {

		if(direct > distance ) {
			//  向下
			scrollTop += perHeight * times;
			if(scrollTop >= maxHeight) {
				scrollTop = maxHeight;
			}
			curNum += 1;
			if(curNum >= maxNum) {
				curNum = maxNum;
			}
		} else {
			// 向上
			scrollTop -= perHeight  * times;
			if(scrollTop <= 0) {
				scrollTop = 0;
			}
			curNum -= 1;
			if(curNum <= 0) {
				curNum = 0;
			}
		}
		ddu_scroll_animate(scrollTop, curNum);
	}

	var selfClickListener = function () {
		// 点击事件 7 5
		$(".btn-after-footer li").click(function() {
			var l = $(this).prevAll().length;
			// 第一页 0
			// 点击第六页，那么前面的元素就是5个
			var num = l +1;
			// l=5 .curNum = 0 num = 6;
			if(curNum >= num) {
				var left = curNum - num + 1;
				curNum = num;
				scroll(0, 1, left);
			} else {
				num = l -1 ;
				var left = l -curNum ;
				curNum = num;
				scroll(0, -1, left)
			}
		});
		//  滚动置顶
		$('.scroll-top').click(function() {
			curNum = 0;
			scrollTop = 0;
			ddu_scroll_animate(scrollTop, curNum);
		});

		// thumbnail
		$("#ddu-desks .thumbnail").hover(function () {
			$(this).children('.desk-detail').css({display:''});
		}, function () {
			$(this).children('.desk-detail').css({display:'none'});
		});
		// events
		$("#ddu-events .thumbnail").hover(function() {
			$(this).children('.event-act').css({display:''});
		}, function () {
			$(this).children('.event-act').css({display:'none'});
		});
	}
	selfClickListener();
	// 自定义鼠标滚动的事件
	var scrollFunc=function(e){
		// 方向
		var direct=0;
		// 兼容的事件
		e=e || window.event;
		e.preventDefault();
		// 向下就是负数，向上就是正数
		var distance = 0;
		if(e.wheelDelta){//IE/Opera/Chrome
			distance =e.wheelDelta;
		}else if(e.detail){//Firefox
			distance =e.detail;
		}
		// 只有完成了，才能继续下一个动画
		if(isFinish) {
			isFinish = false;
			scroll(direct, distance, 1);
		}

	}
	// debug open or not just for console view
	if(!debugCss) {
		//监听鼠标滚动兼容
		if (document.addEventListener) {
			document.addEventListener('DOMMouseScroll', scrollFunc, false);
		}
		// 注册自定义的滚动事件函数到全局
		window.onmousewheel = document.onmousewheel = scrollFunc;
		$('body').css({overflow:'hidden'});
	}
	// 刷新
 	$("body").animate({
 		scrollTop : 0
 	}).swipe({
		swipe: function (event, direction, distance, duration, fingerCount) {

			try {
				distance= 0;
				if (direction == "up") {
					direction = 1;
				} else if (direction == "down") {
					direction = -1;
				}
				// 只有完成了，才能继续下一个动画
				if(isFinish) {
					isFinish = false;
					scroll(direction, distance, 1);
				}
			} catch(err) {
				alert(err.message);
			}

		}
	});;
	keyListener();
});

/**
 * 设置自动滚动
 * @param currentObj
 */
var setScroll = function(currentObj) {
	scrollTop = $(currentObj).parent().parent().parent().height();
	curNum = 1;
	ddu_scroll_animate(scrollTop, curNum);
}
/**
 * 监听Keyboard input
 */
var keyListener = function () {
	window.onkeydown = function (e) {
		e = e || window.event;
		var keycode = e.keyCode;
		// 空格&向下
		if(keycode === 32 || keycode === 40) {
			curNum ++;
			if(curNum >= maxNum) {
				curNum = 0;
			}
			scrollTop = curNum * perHeight;
			ddu_scroll_animate(scrollTop, curNum);
		}
		// 向上
		if(keycode === 38) {
			curNum --;
			scrollTop = curNum * perHeight;
			ddu_scroll_animate(scrollTop, curNum);
		}

		e.preventDefault();
		e.stopPropagation();
		return false;
	}
}

/**
 * ddu 滚动
 * @param scrollTop
 * @param curNum
 */
var ddu_scroll_animate = function(scrollTop, curNum) {
	$('.scroll-top').css({display: 'none'});
	$("footer").css({position:"relative", bottom:"0px"});
	$("body").animate({
		scrollTop : scrollTop
	}, 1000, "swing", function() {
		isFinish = true;
		if(curNum === 0 || curNum === 6) {
			$(".navbar-fixed-top").removeClass('navbar-default');
		} else {
			$(".navbar-fixed-top").addClass("navbar-default");
		}
		if(curNum === maxNum -1) {
			$('.scroll-top').css({display: ''});
			$("footer").css({position:"fixed", bottom:"0px"});
		}
	});
	// 旁边的圆点
	$($(".btn-after-footer li")[curNum]).addClass("hover").siblings().removeClass("hover");
}
/**
 * set bgcolor
 * @param addObjClass
 * @param objClass
 */
var i = 0;
var setBg = function(addObjClass, objClass) {
	var obj = $('.' + objClass);
	if(curNum === 0 || curNum === 6) {
		if (obj.hasClass(addObjClass)) {
			$('.' + objClass).removeClass(addObjClass);
		} else {
			$('.' + objClass).addClass(addObjClass);
		}
	}
};

```
