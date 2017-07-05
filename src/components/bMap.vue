<template>
  <div id='map' class="col-sm-9 col-sm-offset-3 col-md-10 col-md-offset-2 main"></div>
</template>

<script>
  export default {
    props:['list'],
    mounted() {
      this.renderMarker();
    },
    methods:{
      renderMarker() {
        var map = new BMap.Map('map');
        var point = new BMap.Point(123.33740758492789, 41.78797989206016); //这是我家的坐标
        map.centerAndZoom(point, 17);
        map.enableScrollWheelZoom();

        // 生成地图上的小窗口
        var infoWindow = new BMap.InfoWindow("", {
            height: 200
        });
        this.list.forEach(function(place) {

            // 初始化地图标记
            place.marker = new BMap.Marker(new BMap.Point(place.lng, place.lat));

            map.addOverlay(place.marker);

            // 使用百度Place API 获取地点详情
            var ajaxUrl = "http://api.map.baidu.com/place/v2/detail?uid=" + place.id + "&output=json&scope=2&ak=cVicnzEfEuAtITIaDVahjg8i0toBYerK";
            $.ajax({
                url: ajaxUrl,
                type: "GET",
                dataType: "JSONP"
            }).done(function(data) {
                // 判断获取数据状态
                if (data.status === 0) {
                    // 获取地址
                    var address = '地址：' + data.result.address;
                    // 获取图片id（如果有）
                    var picId = data.result.hasOwnProperty('street_id') ? data.result.street_id : '';
                    // 判断是否有详情数据
                    if (data.result.hasOwnProperty('detail_info')) {
                        // 获取标签（如果有）
                        var tag = data.result.detail_info.hasOwnProperty('tag') ? '标签：' + data.result.detail_info.tag : '';
                        // 获取评分（如果有）
                        var rating = data.result.detail_info.hasOwnProperty('overall_rating') ? '评分：' + data.result.detail_info.overall_rating : '';
                        // 获取评价数组（如果有）
                        var reviews = data.result.detail_info.hasOwnProperty('di_review_keyword') ? data.result.detail_info.di_review_keyword : '';
                        // 创建一个空数组，用来将遍历的数组数据转换成字符串
                        var _comment = [];
                        if (reviews != []) {
                            reviews.forEach(function(review) {
                                if (review.hasOwnProperty('keyword')) {
                                    _comment.push(review.keyword);
                                }
                            });
                            if (_comment != []) {
                                var comment = '热评：' + _comment.toString();
                            }
                        }
                        comment = comment ? comment : "";

                    }
                    // 设定小窗口内容
                    var content = '<h4>' + place.title + '<h4>' +
                        '<img alt="街景图" src="http://api.map.baidu.com/panorama/v2?ak=cVicnzEfEuAtITIaDVahjg8i0toBYerK&width=110&height=55&poiid=' + picId + '">' +
                        '<p>' + address + '</p>' +
                        '<p>' + tag + '</p>' +
                        '<p>' + rating + '</p>' +
                        '<p>' + comment + '</p>';

                } else {
                    var content = '<h4>地点数据获取失败，没有这个地点的数据<h4>'; // 若数据获取失败，则设定小窗口内容为错误信息
                }

                // 使用百度EventWrapper开源库来给地图标记设定点击事件

                BMapLib.EventWrapper.addListener(place.marker, "click", function(e) {
                    e.target.openInfoWindow(infoWindow);
                    // 设定动画效果
                    e.target.setAnimation(BMAP_ANIMATION_BOUNCE);
                    setTimeout(function() {
                        e.target.setAnimation(null);
                    }, 500);

                    infoWindow.setContent(content);
                    // 当点击标记时，地图中心会移至标记处
                    map.panTo(e.target.getPosition());
                })

            }).fail(function() {
                alert("地点数据获取失败，请刷新页面重试"); // ajax数据获取失败时的事件
            });
        });
      }
    },
    watch:{
      list() {
        this.renderMarker();
      }
    }
  }
</script>
