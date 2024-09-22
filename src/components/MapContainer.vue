<template>
    <div style='width:28rem;'>
        <label style='color:grey'>根据经纬度获取地址信息</label>
        <div class="input-item">
            <div class="input-item-prepend"><span class="input-item-text">经纬度</span></div>
            <input style="width:500px" id='lnglat' type="text" value='116.39,39.9'>
        </div>
        <div class="input-item">
            <div class="input-item-prepend"><span class="input-item-text">地址</span></div>
            <input style="width:500px" id='address' type="text" disabled>
        </div>
        <input id="regeo" type="button" class="btn" value="经纬度 -> 地址">
    </div>
    <div style='width:28rem;'>
        <label style='color:grey'>根据经纬度获取地址信息</label>
        <div class="input-item">
            <div class="input-item-prepend"><span class="input-item-text">经纬度起点</span></div>
            <input style="width:500px" id='lnglatStart' type="text" value='116.368904,39.913423'>
        </div>
        <div class="input-item">
            <div class="input-item-prepend"><span class="input-item-text">经纬度终点</span></div>
            <input style="width:500px" id='lnglatEnd' type="text" value='116.382122,39.901176'>
        </div>
        <input id="lnglatStartEnd" type="button" class="btn" value="画线">
    </div>
    <div id="container"></div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from "vue";
import AMapLoader from "@amap/amap-jsapi-loader";

let map = null;
onMounted(() => {
    window._AMapSecurityConfig = {
        securityJsCode: "535f73f17b632653974ce7e5af37947e",
    };
    AMapLoader.load({
        key: "f950a5ed7d3503bb722c9383ea2bcb61", // 申请好的Web端开发者Key，首次调用 load 时必填
        version: "2.0", // 指定要加载的 JSAPI 的版本，缺省时默认为 1.4.15
        plugins: ["AMap.Scale", 'AMap.Geocoder', 'AMap.Polyline'], //需要使用的的插件列表，如比例尺'AMap.Scale'，支持添加多个如：['...','...']
    })
        .then((AMap) => {
            map = new AMap.Map("container", {
                // 设置地图容器id
                viewMode: "3D", // 是否为3D地图模式
                zoom: 11, // 初始化地图级别
                center: [116.397428, 39.90923], // 初始化地图中心点位置
            });

            // 根据经纬度搜索地点
            let geocoder = new AMap.Geocoder({
                city: "010", //城市设为北京，默认：“全国”
                radius: 1000 //范围，默认：500
            })

            let marker = new AMap.Marker();
            function regeoCode() {

                let lnglat = document.getElementById('lnglat').value.split(',');
                map.add(marker);
                marker.setPosition(lnglat);

                geocoder.getAddress(lnglat, function (status, result) {
                    if (status === 'complete' && result.regeocode) {
                        let address = result.regeocode.formattedAddress;
                        document.getElementById('address').value = address;
                    } else {
                        log.error('根据经纬度查询地址失败')
                    }
                });
            }

            map.on('click', function (e) {
                document.getElementById('lnglat').value = e.lnglat;
                regeoCode();
            })
            document.getElementById("regeo").onclick = regeoCode;
            document.getElementById('lnglat').onkeydown = function (e) {
                if (e.keyCode === 13) {
                    regeoCode();
                    return false;
                }
                return true;
            };

            // 划两点之间的直线-----------------
            let polylineCopy
            function addPoint() {
                if (polylineCopy) map.remove(polylineCopy);
                let lnglatStart = document.getElementById('lnglatStart').value.split(',');
                let lnglatEnd = document.getElementById('lnglatEnd').value.split(',');

                let path = [
                    new AMap.LngLat(lnglatStart[0], lnglatStart[1]),
                    new AMap.LngLat(lnglatEnd[0], lnglatEnd[1])
                ];
                let polyline = new AMap.Polyline({
                    path: path,
                    strokeWeight: 2, //线条宽度
                    strokeColor: "red", //线条颜色
                    lineJoin: "round", //折线拐点连接处样式
                });
                polylineCopy = polyline
                map.add(polyline);
            }
            document.getElementById("lnglatStartEnd").onclick = addPoint;
        })
        .catch((e) => {
            console.log(e);
        });
});


onUnmounted(() => {
    map?.destroy();
});
</script>

<style scoped>
#container {
    width: 100%;
    height: 800px;
}

.btn {
    width: 10rem;
    margin-left: 6.8rem;
}
</style>
