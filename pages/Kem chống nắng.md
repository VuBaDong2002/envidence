<!-- query Doanh số -->
```sql sum_revenue
SELECT ROUND(
    SUM(revenue_202301 + revenue_202302 + revenue_202303 + revenue_202304 +
        revenue_202305 + revenue_202306), 
    2
) AS sum_revenue
FROM metricda.product
```

<!-- Query sản lượng -->
```sql sum_sale
SELECT
    SUM(sale_202301 + sale_202302 + sale_202303 + sale_202304 +
        sale_202305 + sale_202306) AS sum_sale
FROM metricda.product
```

<!-- Sản Phẩm có lượt bán -->
```sql count_product
SELECT 
    round(COUNT(DISTINCT product_base_id),2) AS product_count
FROM 
    metricda.product
HAVING SUM
    (sale_202301 + sale_202302 + sale_202303 + sale_202304 +
        sale_202305 + sale_202306) > 0;
```

<!-- gian hàng có lượt bán -->
```sql count_shop
SELECT 
    COUNT(DISTINCT shop_base_id) AS shop_count
FROM 
    metricda.product
HAVING SUM
    (sale_202301 + sale_202302 + sale_202303 + sale_202304 +
        sale_202305 + sale_202306) > 0;
```

<!-- Doanh số so với cùng kì năm ngoái -->
```sql growth_rate_revenue
SELECT ((SUM(revenue_202301 + revenue_202302 + revenue_202303 + revenue_202304 +
        revenue_202305 + revenue_202306) - sum(revenue_202201 + revenue_202202 + revenue_202203 + revenue_202204 +
        revenue_202205 + revenue_202206))/ sum(revenue_202201 + revenue_202202 + revenue_202203 + revenue_202204 +
        revenue_202205 + revenue_202206)-1) AS growth_rate_revenue
FROM metricda.product
```
<!-- Sản lượng so với cùng kì năm ngoái -->
```sql growth_rate_sale
SELECT ((SUM(sale_202301 + sale_202302 + sale_202303 + sale_202304 +
        sale_202305 + sale_202306) - sum(sale_202201 + sale_202202 + sale_202203 + sale_202204 +
        sale_202205 + sale_202206))/ sum(sale_202201 + sale_202202 + sale_202203 + sale_202204 +
        sale_202205 + sale_202206)) AS growth_rate_sale
FROM metricda.product
```
<!-- Sản phẩm so với cùng kì năm ngoái  -->
```sql growth_rate_shop_2023
WITH shop_count_2022 AS (
    SELECT 
        COUNT(DISTINCT shop_base_id) AS shop_count
    FROM 
        metricda.product
    HAVING 
        SUM(sale_202201 + sale_202202 + sale_202203 + sale_202204 + sale_202205 + sale_202206) > 0
),
shop_count_2023 AS (
    SELECT 
        COUNT(DISTINCT shop_base_id) AS shop_count
    FROM
        metricda.product
    HAVING
        SUM(sale_202301 + sale_202302 + sale_202303 + sale_202304 + sale_202305 + sale_202306) > 0
)

SELECT
    (shop_count_2023.shop_count - shop_count_2022.shop_count) as growth_rate_shop_2023
FROM
    shop_count_2022,
    shop_count_2023;
```
<!-- Gian hàng có lượt bán so với cùng kì năm ngoái -->
```sql growth_rate_product_2023
with product_count_2023 as (
    select 
        count(distinct product_base_id) as product_count
    from
        metricda.product
    having
        sum(sale_202301 + sale_202302 + sale_202303 + sale_202304 + sale_202305 + sale_202306) > 0
),
product_count_2022 as (
    select 
        count(distinct product_base_id) as product_count
    from
        metricda.product
    having
        sum(sale_202201 + sale_202202 + sale_202203 + sale_202204 + sale_202205 + sale_202206) > 0
)

select 
    (product_count_2023.product_count - product_count_2022.product_count) as growth_rate_product_2023
from
    product_count_2022,
    product_count_2023;
```


<!-- Tỷ trong theo doanh số -->
```sql Floor_weight_by_floor_turnover
WITH sum_revenue AS (
    SELECT 
        SUM(revenue_202301 + revenue_202302 + revenue_202303 + revenue_202304 + revenue_202305 + revenue_202306) AS revenue_sum
    FROM 
        metricda.product
), 

percen_revenue_floor AS (
    SELECT 
        platform_id,
        (SUM(revenue_202301 + revenue_202302 + revenue_202303 + revenue_202304 + revenue_202305 + revenue_202306) / sum_revenue.revenue_sum) * 100 AS revenue_percen
    FROM 
        metricda.product
    CROSS JOIN sum_revenue
    GROUP BY 
        platform_id, sum_revenue.revenue_sum
),
revenue_floor AS (
    SELECT 
        platform_id,
        SUM(revenue_202301 + revenue_202302 + revenue_202303 + revenue_202304 + revenue_202305 + revenue_202306) AS revenue_total
    FROM 
        metricda.product
    GROUP BY 
        platform_id
)

SELECT 
    rf.revenue_total, 
    prf.revenue_percen
FROM
    revenue_floor rf
INNER JOIN 
    percen_revenue_floor prf 
ON 
    rf.platform_id = prf.platform_id
```


```sql fool
select revenue_total as name, revenue_percen as value
from ${Floor_weight_by_floor_turnover}
```

<!-- Tỷ trọng theo sản lượng -->
```sql Floor_sale
WITH sum_sale AS (
    SELECT 
        SUM(sale_202301 + sale_202302 + sale_202303 + sale_202304 + sale_202305 + sale_202306) AS sale_sum
    FROM 
        metricda.product
), 

percen_sale_floor AS (
    SELECT 
        platform_id,
        (SUM(sale_202301 + sale_202302 + sale_202303 + sale_202304 + sale_202305 + sale_202306) / sum_sale.sale_sum) * 100 AS sale_percen
    FROM 
        metricda.product
    CROSS JOIN sum_sale
    GROUP BY 
        platform_id, sum_sale.sale_sum
),
sale_floor AS (
    SELECT 
        platform_id,
        SUM(sale_202301 + sale_202302 + sale_202303 + sale_202304 + sale_202305 + sale_202306) AS sale_total
    FROM 
        metricda.product
    GROUP BY 
        platform_id
)

SELECT 
    rf.sale_total, 
    prf.sale_percen
FROM
    sale_floor rf
INNER JOIN 
    percen_sale_floor prf 
ON 
    rf.platform_id = prf.platform_id
```


```sql fool_sale_
select sale_total as name, sale_percen as value
from ${Floor_sale}
```



<div class="container w-[640px]">
    <div class = "bg-[#E6E6FA] w-[1200px] h-[7px] opacity-[.8]"></div>
    <img src="/static/logo_metric.png" alt="logo_metric" width = "200px"/>
    <p class = "font-['Open_Sans'] text-[18px] mt-[150px] " >BÁO CÁO TỔNG QUAN NGHIÊM CỨU THỊ TRƯỜNG TMDT</p>
    <h1 class = "font-['Open_Sans'] text-[40px] mt-[20px] font-bold">Kem Chống Nắng</h1>

    <div class = "bg-[#000000] w-[200px] h-[2px] mt-[150px] opacity-[.67]"></div>
    <div class= "mt-[10px]">
        <ul>
            <li>
                <p class = "font-['Open_Sans'] text-[14px]">Thực hiện bởi: METRIC.VN </p>
            </li>
            <li>
                <p class = "font-['Open_Sans'] text-[14px]">*Số liệu thống kê từ 01/01/2024 - 30/06/2024 tại sàn Shopee, Lazada và Tiktok Shop</p>
            </li>
        </ul>
    </div>

    <div class = "bg-[#E6E6FA] w-[1200px] h-[10px] mt-[60px] opacity-[.8]"></div>
</div>


<!-- phần giới thiệu  -->
<div class="container mx-auto h-[580px] relative">
    <p class="font-['Open_Sans'] text-[40px] font-bold mt-[60px]">GIỚI THIỆU</p>
    <div class="absolute top-[25%] w-full">
        <div>
            <!-- Mục đích nghiên cứu -->
            <div class="mt-[30px] flex">
                <div class="bg-[#EB8125] w-[30px] h-[8px] mt-[7px]"></div>
                <div class="bg-[#FFFFFF] w-[25px] h-[10px] ml-[10px]"></div>
                <p class="font-['Open_Sans'] text-[16px] text-[#EB8125] font-bold ml-[10px]">MỤC ĐÍCH NGHIÊM CỨU</p>
            </div>
            <div class="ml-[75px] mt-[20px]">
                <ul class="list-disc pl-5 space-y-2">
                    <li><p class="font-['Open_Sans'] text-[14px]">Thống kê số liệu nhóm hàng Kem chống nắng trên sàn TMDT</p></li>
                    <li><p class="font-['Open_Sans'] text-[14px]">Xu hướng phát triển thị trường Kem chống nắng</p></li>
                </ul>
            </div>

            <!-- Phạm vi nghiên cứu -->
            <div class="mt-[30px] flex ">
                <div class="bg-[#EB8125] w-[30px] h-[8px] mt-[7px]"></div>
                <div class="bg-[#FFFFFF] w-[25px] h-[10px] ml-[10px]"></div>
                <p class="font-['Open_Sans'] text-[16px] text-[#EB8125] font-bold ml-[10px]">PHẠM VI NGHIÊM CỨU</p>
            </div>
            <div class="ml-[75px] mt-[20px]">
                <ul class="list-disc pl-5 space-y-2">
                    <li><p class="font-['Open_Sans'] text-[14px]">Thống kê số liệu nhóm hàng Kem chống nắng trên sàn TMDT</p></li>
                    <li><p class="font-['Open_Sans'] text-[14px]">Xu hướng phát triển thị trường Kem chống nắng</p></li>
                </ul>
            </div>
        </div>
    </div>
    <div class="bg-[#E6E6FA] w-full h-[10px] opacity-[.8] absolute bottom-0"></div>
</div>
<!-- Hết phần giới thiệu -->


<!-- Phần nội dung -->
<div class = "ontainer mx-auto h-[580px] relative">
    <p class = "font-['Open_Sans'] text-[40px] mt-[60px] font-bold">NỘI DUNG</p>
    
    <div class = "flex absolute top-[25%] w-full">
        <!-- Phần 01 và 02 -->
        <div class="w-full md:w-1/2 p-[10px]">
            <!-- Phần 01 -->
            <div>
                <div class="flex">
                    <div class="relative">
                        <p class="font-['Open_Sans'] text-[25px] font-bold">01</p>
                        <div class="bg-[#EB8125] w-[25px] h-[8px] absolute top-[80%] left-[0]"></div>
                    </div>
                    <div class="bg-[#FFFFFF] w-[20px] h-[10px]"></div>
                    <p class="font-['Open_Sans'] text-[16px] text-[#EB8125] font-bold  mt-[10px]">TỔNG QUAN THỊ TRƯỜNG</p>
                </div>
                <div>
                    <div class="ml-[45px] mt-[10px]">
                        <ul>
                            <li>
                                <p class = "font-['Open_Sans'] text-[14px]">Cấu trúc quy mô ngành </p>
                            </li>
                            <li>
                                <p class = "font-['Open_Sans'] text-[14px]"> Tổng quan thị trường nhóm hàng </p>
                            </li>
                            <li>
                                <p class = "font-['Open_Sans'] text-[14px]"> Thị trường từng sàn TMDT </p>
                            </li>
                        </ul>
                    </div>
                </div>
            </div>
            <!-- Hết phần 01 -->
            <!-- Phần 02 -->
            <div class = "mt-[20px]">
                <div class="flex">
                    <div class="relative">
                        <p class="font-['Open_Sans'] text-[25px] font-bold">02</p>
                        <div class="bg-[#EB8125] w-[25px] h-[8px] absolute top-[60%] left-[0]"></div>
                    </div>
                    <div class="bg-[#FFFFFF] w-[20px] h-[10px]"></div>
                    <p class="font-['Open_Sans'] text-[16px] text-[#EB8125] font-bold text-wrap w-[300px] mt-[5px]">
                        PHÂN TÍCH THEO PHÂN KHÚC GIÁ, LOẠI & QUY MÔ GIAN HÀNG
                    </p>
                </div>
                <div>
                    <div class="ml-[45px] mt-[10px]">
                        <ul>
                            <li>
                                <p class = "font-['Open_Sans'] text-[14px]">Phân tích theo phân khúc giá</p>
                            </li>
                            <li>
                                <p class = "font-['Open_Sans'] text-[14px]">Phân tích theo kiểu gian hàng</p>
                            </li>
                            <li>
                                <p class = "font-['Open_Sans'] text-[14px]">Phân tích theo quy mô doanh số</p>
                            </li>
                        </ul>
                    </div>
                </div>
            </div>
            <!-- Hết phần 02 -->
        </div>
        <!-- Hết phần 01 và 02 -->
        <!-- Phần 03 và 04 -->
        <div class="w-full md:w-1/2 p-[10px]">
            <!-- Phần 03 -->
            <div>
                <div class="flex">
                    <div class="relative">
                        <p class="font-['Open_Sans'] text-[25px] font-bold">03</p>
                        <div class="bg-[#EB8125] w-[25px] h-[8px] absolute top-[80%] left-[0]"></div>
                    </div>
                    <div class="bg-[#FFFFFF] w-[20px] h-[10px]"></div>
                    <p class="font-['Open_Sans'] text-[16px] text-[#EB8125] font-bold mt-[10px]">
                        PHÂN TÍCH THEO THƯƠNG HIỆU
                    </p>
                </div>
                <div>
                    <div class="ml-[45px] mt-[10px]">
                        <ul>
                            <li>
                                <p class = "font-['Open_Sans'] text-[14px]">Top 10 thương hiệu bán chạy</p>
                            </li>
                            <li>
                                <p class = "font-['Open_Sans'] text-[14px]">Top 05 thương hiệu theo từng sàn</p>
                            </li>
                            <li>
                                <p class = "font-['Open_Sans'] text-[14px]">Top 05 thương hiệu theo phân khúc giá</p>
                            </li>
                        </ul>
                    </div>
                </div>
            </div>
            <!-- Hết phần 01 -->

            <!-- Phần 04 -->
            <div class = "mt-[20px]">
                <div class="flex">
                    <div class="relative">
                        <p class="font-['Open_Sans'] text-[25px] font-bold">04</p>
                        <div class="bg-[#EB8125] w-[25px] h-[8px] absolute top-[60%] left-[0]"></div>
                    </div>
                    <div class="bg-[#FFFFFF] w-[20px] h-[10px]"></div>
                    <div class="flex items-center justify-center">
                        <p class="font-['Open_Sans'] text-[16px] text-[#EB8125] font-bold text-wrap w-[300px] mt-[5px]">
                            PHÂN TÍCH SẢN PHẨM BÁN CHẠY THEO DOANH SỐ
                        </p>
                    </div>
                </div>
                <div>
                    <div class="ml-[45px] mt-[10px]">
                        <ul>
                            <li>
                                <p class = "font-['Open_Sans'] text-[14px]">Danh sách sản phẩm bán chạy trên TMDT</p>
                            </li>
                            <li>
                                <p class = "font-['Open_Sans'] text-[14px]">Danh sách sản phẩm bán chạy theo phân khúc giá</p>
                            </li>
                        </ul>
                    </div>
                </div>
            </div>
        </div>
        <!-- Hết phần 04 -->
        <!-- Hết phần 03 và 04 -->
    </div>

    <!-- Phần 05 -->
    <div class="flex absolute bottom-[20%] p-[10px] ml-[368px]">
        <div class="relative">
            <p class="font-['Open_Sans'] text-[25px] font-bold">05</p>
            <div class="bg-[#EB8125] w-[25px] h-[8px] absolute top-[80%] left-[0]"></div>
        </div>
        <div class="bg-[#FFFFFF] w-[20px] h-[10px]"></div>
        <p class="font-['Open_Sans'] text-[16px] text-[#EB8125] font-bold mt-[10px]">
            KẾT LUẬN
        </p>
    </div>
    <!-- Hết phần 05 -->
    <div class="bg-[#E6E6FA] w-full h-[10px] opacity-[.8] absolute bottom-0"></div>
</div>
<!-- Hết phần nội dung-->

<!-- <div class = "bg-[#E6E6FA] w-[1200px] h-[7px] opacity-[.8]"></div> -->
<div class = "container mx-auto h-[640px] relative">
    <div class = "flex">
        <p class= "font-['Open_Sans'] text-[12px] mr-[10px]">Tổng quan thị trường</p>
        <p class = "font-['Open_Sans'] text-[12px] mr-[10px]"> => </p>
        <p class= "font-['Open_Sans'] text-[12px]">Nhóm hàng Kem chống nắng</p>
    </div>

    <h1 class = "font-['Open_Sans'] text-[30px] font-bold">Nhóm hàng Kem chống nắng</h1>
    <div class="bg-[#EB8125] w-[50px] h-[8px]"></div>

    <div class = "flex mt-[25px]">
        <div class = "w-[10px] h-[25px] bg-[#FDD5C3] rounded-[12px] mr-[10px]"></div>
        <p class= "font-['Open Sans SemiBold'] text-[16px] mr-[10px]">Tổng quan thị trường</p>
    </div>

    <!-- Doanh số, sản lượng, sản phẩm có lượt bán, gian hàng có lượt bán -->
    <div class="flex justify-center mt-[20px]">
        <!-- Doanh số -->
        <div class="w-[24%] m-[5px] bg-[rgba(230,230,230,0.5)] rounded-[12px]">
            <p class="text-[13px] font-bold text-center mt-[5px]">DOANH SỐ</p>
            <p class = "font-bold text-center mt-[5px]"><Value data={sum_revenue} fmt="num0"/></p>
            <div class = "flex mt-[5px] justify-center mb-[5px]">
                <Delta data={growth_rate_revenue} symbolPosition=left fmt = "pct"/>
                <p class = " ml-[2px] text-[12px] text-[#006400] mt-[5px]"> so với cùng kỳ</p>
            </div>
        </div>
        <!-- sản lượng -->
        <div class="w-[24%] m-[5px] bg-[rgba(230,230,230,0.5)] rounded-[12px]">
            <p class="text-[13px] font-bold text-center mt-[5px] ">SẢN LƯỢNG</p>
            <p class = "font-bold text-center mt-[5px]"><Value data={sum_sale} fmt = "num0"/></p>
            <div class = "flex mt-[5px] justify-center mb-[5px]">
                <Delta data={growth_rate_sale} symbolPosition=left fmt = "pct"/>
                <p class = " ml-[2px] text-[12px] text-[#006400] mt-[5px]"> so với cùng kỳ</p>
            </div>
        </div>
        <!-- Sản phẩm có lượt bán -->
        <div class= "w-[24%] m-[5px] bg-[rgba(230,230,230,0.5)] rounded-[12px]">
            <p class="text-[13px] font-bold text-center mt-[5px]">SẢN PHẨM CÓ LƯỢT BÁN</p>
            <p class = "font-bold text-center mt-[5px]"><Value data={count_product} fmt = "num0"/></p>
            <div class = "flex mt-[5px] justify-center mb-[5px]">
                <Delta data={growth_rate_product_2023} symbolPosition=left fmt = "pct"/>
                <p class = " ml-[2px] text-[12px] text-[#006400] mt-[5px]"> so với cùng kỳ</p>
            </div>
        </div>
        <!-- Gian hàng có lượt bán -->
        <div class="w-[24%] m-[5px] bg-[rgba(230,230,230,0.5)] rounded-[12px]">
            <p class="text-[13px] font-bold text-center mt-[5px]">GIAN HÀNG CÓ LƯỢT BÁN</p>
            <p class = "font-bold text-center mt-[5px]"><Value data={count_shop} fmt = "num0"/></p>
            <div class = "flex mt-[5px] justify-center mb-[5px]">
                <Delta data={growth_rate_shop_2023} symbolPosition=left fmt = "pct"/>
                <p class = " ml-[2px] text-[12px] text-[#006400] mt-[5px]"> so với cùng kỳ</p>
            </div>
        </div>
    </div>

    <div>
        <!-- Biểu đồ doanh số và sản lượng -->
        <div class = "flex">
            <!-- Biểu đồ doanh số -->
            <div  class = "w-[50%]">
                <div class = "flex mt-[25px]">
                    <div class = "w-[10px] h-[20px] bg-[#FDD5C3] rounded-[12px] mr-[10px]"></div>
                    <p class= "font-['Open Sans SemiBold'] text-[16px] mr-[10px]">Tỷ trọng theo doanh số</p>
                </div>
                <div>
                    <ECharts 
                    config={{
                        tooltip: {
                            formatter: '{b}: {c} ({d}%)'
                        },
                        series: [
                        {
                            type: 'pie',
                            data: fool,
                            radius: ['40%', '65%'],
                            itemStyle: {
                                color: function(params) {
                                    const colorList = ['#F95F26', '#000000', '#21B8F0', '#2F5597'];
                                    return colorList[params.dataIndex % colorList.length];
                                },
                            },
                            label: {
                                formatter: function(params) {
                                    if (params.name.length < 9) {
                                        return (Math.round(params.name / 1000000 * 100) / 100) + 'M'; 
                                    } else if  (params.name.length > 9) {
                                        return (Math.round(params.name / 1000000000 * 100) / 100) + 'B';
                                    }
                                }
                            }
                        },
                        {
                            type: 'pie',
                            data: fool,
                            radius: ['40%', '65%'],
                            itemStyle: {
                                color: function(params) {
                                    const colorList = ['#F95F26', '#000000', '#21B8F0', '#2F5597'];
                                    return colorList[params.dataIndex % colorList.length];
                                }
                            },
                            label: {
                                formatter: function(params) {
                                    return Math.round(params.value * 100) / 100 + '%';
                                },
                                position: 'inner',
                                fontSize: 8
                            }
                        }
                        ]
                    }}/>
                </div>
            </div>
            <!-- Biểu đồ sản lượng -->
            <div  class = "w-[50%]">
                <div class = "flex mt-[25px]">
                    <div class = "w-[10px] h-[20px] bg-[#FDD5C3] rounded-[12px] mr-[10px]"></div>
                    <p class= "font-['Open Sans SemiBold'] text-[16px] mr-[10px]">Tỷ trọng theo sản lượng</p>
                </div>
                <div>
                <ECharts 
                    config={{
                        tooltip: {
                            formatter: '{b}: {c} ({d}%)'
                        },
                        series: [
                        {
                            type: 'pie',
                            data: fool_sale_,
                            radius: ['40%', '65%'],
                            itemStyle: {
                                color: function(params) {
                                    const colorList = ['#F95F26', '#000000', '#21B8F0', '#2F5597'];
                                    return colorList[params.dataIndex % colorList.length];
                                },
                            },
                            label: {
                                formatter: function(params) {
                                    if (params.name.length < 9) {
                                        return (Math.round(params.name / 1000000 * 100) / 100) + 'M'; 
                                    } else if  (params.name.length > 9) {
                                        return (Math.round(params.name / 1000000000 * 100) / 100) + 'B';
                                    }
                                }
                            }
                        },
                        {
                            type: 'pie',
                            data: fool_sale_,
                            radius: ['40%', '65%'],
                            itemStyle: {
                                color: function(params) {
                                    const colorList = ['#F95F26', '#000000', '#21B8F0', '#2F5597'];
                                    return colorList[params.dataIndex % colorList.length];
                                }
                            },
                            label: {
                                formatter: function(params) {
                                    return Math.round(params.value * 100) / 100 + '%';
                                },
                                position: 'inner',
                                fontSize: 8
                            }
                        }
                        ]
                    }}/>
                </div>
            </div>
        </div>
    </div>

    <!-- Chú thích -->
    <div class = "flex justify-center">
        <!-- shopee -->
        <div class = "flex mr-[10px]">
            <div class = "h-[10px] w-[10px] bg-[#F95F26] mr-[5px] mt-[3px]"></div>
            <p class = "font-['Open Sans SemiBold'] text-[12px]">Shopee</p>
        </div>
        <!-- lazada -->
        <div class = "flex mr-[10px]">
            <div class = "h-[10px] w-[10px] bg-[#000000] mr-[5px] mt-[3px]"></div>
            <p class = "font-['Open Sans SemiBold'] text-[12px]">Lazada</p>
        </div>
        <!-- tiktok -->
        <div class = "flex mr-[10px]">
            <div class = "h-[10px] w-[10px] bg-[#21B8F0] mr-[5px] mt-[3px]"></div>
            <p class = "font-['Open Sans SemiBold'] text-[12px]">Tiktok</p>
        </div>
        <!-- tiki -->
        <div class = "flex mr-[10px]">
            <div class = "h-[10px] w-[10px] bg-[#2F5597] mr-[5px] mt-[3px]"></div>
            <p class = "font-['Open Sans SemiBold'] text-[12px]">Tiki</p>
        </div>
    </div>
    <div class="bg-[#E6E6FA] w-full h-[10px] opacity-[.8] absolute bottom-0"></div>
</div>

```sql san_luong_theo_thang
WITH PlatformData AS (
    SELECT
        platform_id,
        CASE
            WHEN platform_id = 1 THEN 'Shopee'
            WHEN platform_id = 8 THEN 'Tiktok shop'
            WHEN platform_id = 2 THEN 'Lazada'
            WHEN platform_id = 3 THEN 'Tiki'
        END AS platform_name,
        SUM(sale_202301) AS sale_01,
        SUM(sale_202302) AS sale_02,
        SUM(sale_202303) AS sale_03,
        SUM(sale_202304) AS sale_04,
        SUM(sale_202305) AS sale_05,
        SUM(sale_202306) AS sale_06,
        SUM(sale_202307) AS sale_07,
        SUM(sale_202308) AS sale_08,
        SUM(sale_202309) AS sale_09,
        SUM(sale_202310) AS sale_10,
        SUM(sale_202311) AS sale_11,
        SUM(sale_202312) AS sale_12,
        CASE
            WHEN platform_id = 1 THEN 1  
            WHEN platform_id = 8 THEN 2 
            WHEN platform_id = 2 THEN 3  
            WHEN platform_id = 3 THEN 4 
        END AS platform_order
    FROM metricda.product
    GROUP BY platform_id
),
SalesData AS (
    SELECT platform_name, 1 AS month, sale_01 AS sale, platform_order FROM PlatformData
    UNION ALL
    SELECT platform_name, 2 AS month, sale_02 AS sale, platform_order FROM PlatformData
    UNION ALL
    SELECT platform_name, 3 AS month, sale_03 AS sale, platform_order FROM PlatformData
    UNION ALL
    SELECT platform_name, 4 AS month, sale_04 AS sale, platform_order FROM PlatformData
    UNION ALL
    SELECT platform_name, 5 AS month, sale_05 AS sale, platform_order FROM PlatformData
    UNION ALL
    SELECT platform_name, 6 AS month, sale_06 AS sale, platform_order FROM PlatformData
    UNION ALL
    SELECT platform_name, 7 AS month, sale_07 AS sale, platform_order FROM PlatformData
    UNION ALL
    SELECT platform_name, 8 AS month, sale_08 AS sale, platform_order FROM PlatformData
    UNION ALL
    SELECT platform_name, 9 AS month, sale_09 AS sale, platform_order FROM PlatformData
    UNION ALL
    SELECT platform_name, 10 AS month, sale_10 AS sale, platform_order FROM PlatformData
    UNION ALL
    SELECT platform_name, 11 AS month, sale_11 AS sale, platform_order FROM PlatformData
    UNION ALL
    SELECT platform_name, 12 AS month, sale_12 AS sale, platform_order FROM PlatformData
)
SELECT * 
FROM SalesData
ORDER BY platform_order, month
```

```sql doanh_thu_theo_thang
WITH PlatformData AS (
    SELECT
        platform_id,
        CASE
            WHEN platform_id = 1 THEN 'Shopee'
            WHEN platform_id = 8 THEN 'Tiktok shop'
            WHEN platform_id = 2 THEN 'Lazada'
            WHEN platform_id = 3 THEN 'Tiki'
        END AS platform_name,
        SUM(revenue_202301) AS revenue_01,
        SUM(revenue_202302) AS revenue_02,
        SUM(revenue_202303) AS revenue_03,
        SUM(revenue_202304) AS revenue_04,
        SUM(revenue_202305) AS revenue_05,
        SUM(revenue_202306) AS revenue_06,
        SUM(revenue_202307) AS revenue_07,
        SUM(revenue_202308) AS revenue_08,
        SUM(revenue_202309) AS revenue_09,
        SUM(revenue_202310) AS revenue_10,
        SUM(revenue_202311) AS revenue_11,
        SUM(revenue_202312) AS revenue_12,
        CASE
            WHEN platform_id = 1 THEN 1  -- Shopee
            WHEN platform_id = 8 THEN 2  -- Tiktok shop
            WHEN platform_id = 2 THEN 3  -- Lazada
            WHEN platform_id = 3 THEN 4  -- Tiki
        END AS platform_order
    FROM metricda.product
    GROUP BY platform_id
),
RevenueData AS (
    SELECT platform_name, 1 AS month, revenue_01 AS revenue, platform_order FROM PlatformData
    UNION ALL
    SELECT platform_name, 2 AS month, revenue_02 AS revenue, platform_order FROM PlatformData
    UNION ALL
    SELECT platform_name, 3 AS month, revenue_03 AS revenue, platform_order FROM PlatformData
    UNION ALL
    SELECT platform_name, 4 AS month, revenue_04 AS revenue, platform_order FROM PlatformData
    UNION ALL
    SELECT platform_name, 5 AS month, revenue_05 AS revenue, platform_order FROM PlatformData
    UNION ALL
    SELECT platform_name, 6 AS month, revenue_06 AS revenue, platform_order FROM PlatformData
    UNION ALL
    SELECT platform_name, 7 AS month, revenue_07 AS revenue, platform_order FROM PlatformData
    UNION ALL
    SELECT platform_name, 8 AS month, revenue_08 AS revenue, platform_order FROM PlatformData
    UNION ALL
    SELECT platform_name, 9 AS month, revenue_09 AS revenue, platform_order FROM PlatformData
    UNION ALL
    SELECT platform_name, 10 AS month, revenue_10 AS revenue, platform_order FROM PlatformData
    UNION ALL
    SELECT platform_name, 11 AS month, revenue_11 AS revenue, platform_order FROM PlatformData
    UNION ALL
    SELECT platform_name, 12 AS month, revenue_12 AS revenue, platform_order FROM PlatformData
)
SELECT * 
FROM RevenueData
ORDER BY platform_order, month;
```

<div class = "container mx-auto h-[640px] relative">
    <div class = "flex">
        <p class= "font-['Open_Sans'] text-[12px] mr-[10px]">Tổng quan thị trường</p>
        <p class = "font-['Open_Sans'] text-[12px] mr-[10px]"> => </p>
        <p class= "font-['Open_Sans'] text-[12px]">Nhóm hàng Kem chống nắng</p>
    </div>

    <h1 class = "font-['Open_Sans'] text-[30px] font-bold">Nhóm hàng Kem chống nắng</h1>
    <div class="bg-[#EB8125] w-[50px] h-[8px]"></div>

    <div class = "flex mt-[25px]">
        <div class = "w-[10px] h-[25px] bg-[#FDD5C3] rounded-[12px] mr-[10px]"></div>
        <p class= "font-['Open Sans SemiBold'] text-[16px] mr-[10px]">Tăng trưởng doanh thu theo tháng</p>
    </div>

    <div class = "w-[640px]">
    <LineChart 
    data={san_luong_theo_thang} 
    title="Tăng trưởng sản lượng theo tháng"
    x="month" 
    y= "sale"  
    xAxisTitle="Tháng"
    yAxisTitle="Sản lượng"
    markers = true
    series="platform_name"  
    colorPalette={
    ['#FF5722', '#000022','#104E8B', '#00BFFF']  
}
/>
    </div>

    <div class = "flex mt-[25px]">
        <div class = "w-[10px] h-[25px] bg-[#FDD5C3] rounded-[12px] mr-[10px]"></div>
        <p class= "font-['Open Sans SemiBold'] text-[16px] mr-[10px]">Doanh thu theo từng tháng</p>
    </div>

    <div class = "w-[640px]">
    <LineChart 
    data={doanh_thu_theo_thang} 
    title="Tăng trưởng doanh thu theo tháng"
    x="month" 
    y= "revenue"  
    xAxisTitle="Tháng"
    yAxisTitle="Doanh thu"
    markers = true
    series="platform_name"  
    colorPalette={
    ['#FF5722', '#000022','#104E8B', '#00BFFF']
    
}
/>
    </div>
</div>

# Phân tích thương hiệu

## Top Thương hiệu theo Doanh số

<!-- Top 10 Thương hiệu theo doanh số -->

```sql top_revenue_10_brand_theo_san
WITH total_revenue_per_brand AS (
    SELECT
        cleaned_brand,
        SUM(
            revenue_202301 + revenue_202302 + revenue_202303 +
            revenue_202304 + revenue_202305 + revenue_202306 +
            revenue_202307 + revenue_202308 + revenue_202309 +
            revenue_202310 + revenue_202311 + revenue_202312
        ) AS total_revenue
    FROM
        metricda.product
    WHERE
        cleaned_brand NOT IN ('chưa biết', 'no brand')
    GROUP BY
        cleaned_brand
),
top_10_brands AS (
    SELECT
        cleaned_brand,
        total_revenue
    FROM
        total_revenue_per_brand
    ORDER BY
        total_revenue DESC
    LIMIT 10
),
revenue_per_platform AS (
    SELECT
        k.platform_id,
        k.cleaned_brand,
        SUM(
            k.revenue_202301 + k.revenue_202302 + k.revenue_202303 +
            k.revenue_202304 + k.revenue_202305 + k.revenue_202306 +
            k.revenue_202307 + k.revenue_202308 + k.revenue_202309 +
            k.revenue_202310 + k.revenue_202311 + k.revenue_202312
        ) AS platform_revenue
    FROM
        metricda.product k
    JOIN
        top_10_brands t10 ON k.cleaned_brand = t10.cleaned_brand
    WHERE
        k.cleaned_brand NOT IN ('chưa biết', 'no brand')
    GROUP BY
        k.platform_id,
        k.cleaned_brand
),
platform_mapping AS (
    SELECT 1 AS platform_id, 'Shopee' AS platform_name, 1 AS sort_order
    UNION ALL
    SELECT 2, 'Lazada', 2
    UNION ALL
    SELECT 8, 'Tiktok', 3
    UNION ALL
    SELECT 3, 'Tiki', 4
)

SELECT
    t10.cleaned_brand,
    t10.total_revenue,
    COALESCE(pm.platform_name, CONCAT('Platform_', CAST(rp.platform_id AS VARCHAR))) AS platform_name,
    rp.platform_revenue
FROM
    top_10_brands t10
LEFT JOIN
    revenue_per_platform rp ON t10.cleaned_brand = rp.cleaned_brand
LEFT JOIN
    platform_mapping pm ON rp.platform_id = pm.platform_id
ORDER BY
    t10.total_revenue DESC,
    pm.sort_order;
```


<BarChart 
    data={top_revenue_10_brand_theo_san}
    x=cleaned_brand
    y=platform_revenue
    series=platform_name
    swapXY=true
    title="Top 10 Thương hiệu theo Doanh số"
    colorPalette={
    ['#FF5722', '#000022','#104E8B', '#00BFFF']}
    yAxisTitle = 'Tỷ đồng'
    xAxisTitle = 'Thương hiệu'
    yFmt='#,##0.00,,,'
/>


<div class="bg-[#FFFFFF] w-[20px] h-[170px]"></div>

<!-- Top brand sàn Shopee -->

```sql top_5_brand_shopee_theo_revenue
WITH CURRENT_PERIOD AS (
    SELECT 
        cleaned_brand,
        SUM(
            revenue_202301 + revenue_202302 + revenue_202303 +
            revenue_202304 + revenue_202305 + revenue_202306 +
            revenue_202307 + revenue_202308 + revenue_202309 +
            revenue_202310 + revenue_202311 + revenue_202312
        ) AS total_revenue
    FROM 
        metricda.product
    WHERE 
        platform_id = 1
        AND cleaned_brand NOT IN ('chưa biết', 'no brand')
    GROUP BY 
        cleaned_brand
)
SELECT 
    cleaned_brand,
    total_revenue
FROM 
    CURRENT_PERIOD
ORDER BY 
    total_revenue DESC
LIMIT 5;
```

<!-- Top Sản phẩm sàn Lazada -->
```sql top_5_brand_lazada_theo_revenue
WITH CURRENT_PERIOD AS (
    SELECT 
        cleaned_brand,
        SUM(
            revenue_202301 + revenue_202302 + revenue_202303 +
            revenue_202304 + revenue_202305 + revenue_202306 +
            revenue_202307 + revenue_202308 + revenue_202309 +
            revenue_202310 + revenue_202311 + revenue_202312
        ) AS total_revenue
    FROM 
        metricda.product
    WHERE 
        platform_id = 2
        AND cleaned_brand NOT IN ('chưa biết', 'no brand')
    GROUP BY 
        cleaned_brand
)
SELECT 
    cleaned_brand,
    total_revenue
FROM 
    CURRENT_PERIOD
ORDER BY 
    total_revenue DESC
LIMIT 5;
```

<!-- Top Sản phẩm sàn Tiktok -->
```sql top_5_brand_tiktok_theo_revenue
WITH CURRENT_PERIOD AS (
    SELECT 
        cleaned_brand,
        SUM(
            revenue_202301 + revenue_202302 + revenue_202303 +
            revenue_202304 + revenue_202305 + revenue_202306 +
            revenue_202307 + revenue_202308 + revenue_202309 +
            revenue_202310 + revenue_202311 + revenue_202312
        ) AS total_revenue
    FROM 
        metricda.product
    WHERE 
        platform_id = 8
        AND cleaned_brand NOT IN ('chưa biết', 'no brand')
    GROUP BY 
        cleaned_brand
)
SELECT 
    cleaned_brand,
    total_revenue
FROM 
    CURRENT_PERIOD
ORDER BY 
    total_revenue DESC
LIMIT 5;
```


<!-- Top Sản phẩm sàn tiki -->
```sql top_5_brand_tiki_theo_revenue
WITH CURRENT_PERIOD AS (
    SELECT 
        cleaned_brand,
        SUM(
            revenue_202301 + revenue_202302 + revenue_202303 +
            revenue_202304 + revenue_202305 + revenue_202306 +
            revenue_202307 + revenue_202308 + revenue_202309 +
            revenue_202310 + revenue_202311 + revenue_202312
        ) AS total_revenue
    FROM 
        metricda.product
    WHERE 
        platform_id = 3
        AND cleaned_brand NOT IN ('chưa biết', 'no brand')
    GROUP BY 
        cleaned_brand
)
SELECT 
    cleaned_brand,
    total_revenue
FROM 
    CURRENT_PERIOD
ORDER BY 
    total_revenue DESC
LIMIT 5;
```


<Grid cols=2 >
<BarChart 
    data={top_5_brand_shopee_theo_revenue}
    x=cleaned_brand
    y=total_revenue
    swapXY=true
    yFmt='#,##0.00,,,'
    labelSize = 10
    labels=true
    title="Top Thương hiệu trên sàn Shopee"
    showAllLabels = true
    yAxisTitle = 'Tỷ đồng'
    xAxisTitle = 'Thương hiệu'
    colorPalette='#FF5722'
/>




<BarChart 
    data={top_5_brand_lazada_theo_revenue}
    x=cleaned_brand
    y=total_revenue
    swapXY=true
    yFmt='#,##0.00,,,'
    labels=true
    labelSize = 10
    title="Top Thương hiệu trên sàn Lazada"
    showAllLabels = true
    yAxisTitle = 'Tỷ đồng'
    xAxisTitle = 'Thương hiệu'
    colorPalette='#104E8B'
/>



<BarChart 
    data={top_5_brand_tiktok_theo_revenue}
    x=cleaned_brand
    y=total_revenue
    swapXY=true
    yFmt='#,##0.00,,,'
    labelSize = 10
    labels=true
    title="Top Thương hiệu trên sàn Tiktok"
    showAllLabels = true
    yAxisTitle = 'Tỷ đồng'
    xAxisTitle = 'Thương hiệu'
    colorPalette='#000022'
/>



<BarChart 
    data={top_5_brand_tiki_theo_revenue}
    x=cleaned_brand
    y=total_revenue
    swapXY=true
    yFmt='#,##0.00,,,'
    labels=true
    labelSize = 10
    title="Top Thương hiệu trên sàn Tiki"
    showAllLabels = true
    yAxisTitle = 'Tỷ đồng'
    xAxisTitle = 'Thương hiệu'
    colorPalette='#00BFFF'
/>
</Grid>

<div class="bg-[#FFFFFF] w-[20px] h-[250px]"></div>

## Top Thương hiệu theo Sản lượng

```sql top_sale_10_brand_theo_san
WITH total_sale_per_brand AS (
    SELECT
        cleaned_brand,
        SUM(
            sale_202301 + sale_202302 + sale_202303 +
            sale_202304 + sale_202305 + sale_202306 +
            sale_202307 + sale_202308 + sale_202309 +
            sale_202310 + sale_202311 + sale_202312
        ) AS total_sale
    FROM
        metricda.product
    WHERE
        cleaned_brand NOT IN ('chưa biết', 'no brand')
    GROUP BY
        cleaned_brand
),
top_10_brands AS (
    SELECT
        cleaned_brand,
        total_sale
    FROM
        total_sale_per_brand
    ORDER BY
        total_sale DESC
    LIMIT 10
),
sale_per_platform AS (
    SELECT
        k.platform_id,
        k.cleaned_brand,
        SUM(
            k.sale_202301 + k.sale_202302 + k.sale_202303 +
            k.sale_202304 + k.sale_202305 + k.sale_202306 +
            k.sale_202307 + k.sale_202308 + k.sale_202309 +
            k.sale_202310 + k.sale_202311 + k.sale_202312
        ) AS platform_sale
    FROM
        metricda.product k
    JOIN
        top_10_brands t10 ON k.cleaned_brand = t10.cleaned_brand
    WHERE
        k.cleaned_brand NOT IN ('chưa biết', 'no brand')
    GROUP BY
        k.platform_id,
        k.cleaned_brand
),
platform_mapping AS (
    SELECT 1 AS platform_id, 'Shopee' AS platform_name, 1 AS sort_order
    UNION ALL
    SELECT 2, 'Lazada', 2
    UNION ALL
    SELECT 8, 'Tiktok', 3
    UNION ALL
    SELECT 3, 'Tiki', 4
)

SELECT
    t10.cleaned_brand,
    t10.total_sale,
    COALESCE(pm.platform_name, CONCAT('Platform_', CAST(rp.platform_id AS VARCHAR))) AS platform_name,
    rp.platform_sale
FROM
    top_10_brands t10
LEFT JOIN
    sale_per_platform rp ON t10.cleaned_brand = rp.cleaned_brand
LEFT JOIN
    platform_mapping pm ON rp.platform_id = pm.platform_id
ORDER BY
    t10.total_sale DESC,
    pm.sort_order;
```





<!-- Top brand sàn Shopee -->

```sql top_5_brand_shopee_theo_sale
WITH CURRENT_PERIOD AS (
    SELECT 
        cleaned_brand,
        SUM(
            sale_202301 + sale_202302 + sale_202303 +
            sale_202304 + sale_202305 + sale_202306 +
            sale_202307 + sale_202308 + sale_202309 +
            sale_202310 + sale_202311 + sale_202312
        ) AS total_sale
    FROM 
        metricda.product
    WHERE 
        platform_id = 1
        AND cleaned_brand NOT IN ('chưa biết', 'no brand')
    GROUP BY 
        cleaned_brand
)
SELECT 
    cleaned_brand,
    total_sale
FROM 
    CURRENT_PERIOD
ORDER BY 
    total_sale DESC
LIMIT 5;
```





<!-- Top Sản phẩm sàn Lazada -->
```sql top_5_brand_lazada_theo_sale
WITH CURRENT_PERIOD AS (
    SELECT 
        cleaned_brand,
        SUM(
            sale_202301 + sale_202302 + sale_202303 +
            sale_202304 + sale_202305 + sale_202306 +
            sale_202307 + sale_202308 + sale_202309 +
            sale_202310 + sale_202311 + sale_202312
        ) AS total_sale
    FROM 
        metricda.product
    WHERE 
        platform_id = 2
        AND cleaned_brand NOT IN ('chưa biết', 'no brand')
    GROUP BY 
        cleaned_brand
)
SELECT 
    cleaned_brand,
    total_sale
FROM 
    CURRENT_PERIOD
ORDER BY 
    total_sale DESC
LIMIT 5;
```





<!-- Top Sản phẩm sàn Tiktok -->
```sql top_5_brand_tiktok_theo_sale
WITH CURRENT_PERIOD AS (
    SELECT 
        cleaned_brand,
        SUM(
            sale_202301 + sale_202302 + sale_202303 +
            sale_202304 + sale_202305 + sale_202306 +
            sale_202307 + sale_202308 + sale_202309 +
            sale_202310 + sale_202311 + sale_202312
        ) AS total_sale
    FROM 
        metricda.product
    WHERE 
        platform_id = 8
        AND cleaned_brand NOT IN ('chưa biết', 'no brand')
    GROUP BY 
        cleaned_brand
)
SELECT 
    cleaned_brand,
    total_sale
FROM 
    CURRENT_PERIOD
ORDER BY 
    total_sale DESC
LIMIT 5;
```

<!-- Top Sản phẩm sàn tiki -->
```sql top_5_brand_tiki_theo_sale
WITH CURRENT_PERIOD AS (
    SELECT 
        cleaned_brand,
        SUM(
            sale_202301 + sale_202302 + sale_202303 +
            sale_202304 + sale_202305 + sale_202306 +
            sale_202307 + sale_202308 + sale_202309 +
            sale_202310 + sale_202311 + sale_202312
        ) AS total_sale
    FROM 
        metricda.product
    WHERE 
        platform_id = 3
        AND cleaned_brand NOT IN ('chưa biết', 'no brand')
    GROUP BY 
        cleaned_brand
)
SELECT 
    cleaned_brand,
    total_sale
FROM 
    CURRENT_PERIOD
ORDER BY 
    total_sale DESC
LIMIT 5;
```


<BarChart 
    data={top_sale_10_brand_theo_san}
    x=cleaned_brand
    y=platform_sale
    series=platform_name
    swapXY=true
    labelSize = 10
    title="Top 10 Thương hiệu theo Sản lượng"
    colorPalette={
    ['#FF5722','#104E8B', '#000022', '#00BFFF']}
    yAxisTitle = 'Nghìn sản phẩm'
    xAxisTitle = 'Thương hiệu'
    yFmt="#,##0.00,"
/>


<div class="bg-[#FFFFFF] w-[20px] h-[150px]"></div>

<Grid cols=2>

<BarChart 
    data={top_5_brand_shopee_theo_sale}
    x=cleaned_brand
    y=total_sale
    swapXY=true
    labels=true
    title="Top Thương hiệu trên sàn Shopee"
    showAllLabels = true
    yAxisTitle = 'Nghìn sản phẩm'
    xAxisTitle = 'Thương hiệu'
    yFmt="#,##0.00,"
    colorPalette='#FF5722'
/>
<BarChart 
    data={top_5_brand_lazada_theo_sale}
    x=cleaned_brand
    y=total_sale
    swapXY=true
    labels=true
    title="Top Thương hiệu trên sàn Lazada"
    showAllLabels = true
    yAxisTitle = 'Nghìn sản phẩm'
    xAxisTitle = 'Thương hiệu'
    yFmt="#,##0.00,"
    colorPalette='#104E8B'
/>

<BarChart 
    data={top_5_brand_tiktok_theo_sale}
    x=cleaned_brand
    y=total_sale
    swapXY=true
    labels=true
    title="Top Thương hiệu trên sàn Tiktok"
    showAllLabels = true
    yAxisTitle = 'Nghìn sản phẩm'
    xAxisTitle = 'Thương hiệu'
    yFmt="#,##0.00,"
    colorPalette='#000022'
/>


<BarChart 
    data={top_5_brand_tiki_theo_sale}
    x=cleaned_brand
    y=total_sale
    swapXY=true
    labels=true
    title="Top Thương hiệu trên sàn Tiki"
    showAllLabels = true
    yAxisTitle = 'Nghìn sản phẩm'
    xAxisTitle = 'Thương hiệu'
    yFmt="#,##0.00,"
    colorPalette='#00BFFF'
/>

</Grid>


<div class="bg-[#FFFFFF] w-[20px] h-[200px]"></div>

# Top sản phẩm theo Doanh số
## Top sản phẩm toàn sàn
```sql top_product
select product_name,
       (revenue_202301 + revenue_202302 + revenue_202303 + revenue_202304 + revenue_202305 + revenue_202306 +
            revenue_202307 + revenue_202308 + revenue_202309 + revenue_202310 + revenue_202311 + revenue_202312) as revenue,
       concat('<img src="', url_thumbnail,'" width="300px" />') as thumbnail,
       platform_id,
       price,
       cleaned_brand 
from metricda.product
order by (revenue_202301 + revenue_202302 + revenue_202303 + revenue_202304 + revenue_202305 + revenue_202306 +
            revenue_202307 + revenue_202308 + revenue_202309 + revenue_202310 + revenue_202311 + revenue_202312) desc nulls last
limit 5
```

<DataTable data={top_product} wrapTitles=true subtotals=true totalRow=true groupType=section>
    <Column id=thumbnail contentType=html/>
    	<Column id=product_name wrap=true/>
    	<Column id=revenue title="Revenue (tỷ đồng)" fmt= '#,##0.00,,,'  contentType=colorscale/>
    	<Column id=platform_id/>
    	<Column id=price title="Giá (Nghìn đồng)"  fmt = "#,##0.00," />
    	<Column id=cleaned_brand title="Thương hiệu"/>
      
</DataTable>

<div class="bg-[#FFFFFF] w-[20px] h-[350px]"></div>

## Phân khúc giá dưới 100K
```sql pkg_100
select product_name,
       (revenue_202301 + revenue_202302 + revenue_202303 + revenue_202304 + revenue_202305 + revenue_202306 +
            revenue_202307 + revenue_202308 + revenue_202309 + revenue_202310 + revenue_202311 + revenue_202312) as revenue,
       concat('<img src="', url_thumbnail,'" width="300px" />') as thumbnail,
       platform_id,
       price,
       cleaned_brand 
from metricda.product
where price < 100000
        and cleaned_brand not in ('chưa biết', 'no brand')
order by (revenue_202301 + revenue_202302 + revenue_202303 + revenue_202304 + revenue_202305 + revenue_202306 +
            revenue_202307 + revenue_202308 + revenue_202309 + revenue_202310 + revenue_202311 + revenue_202312) desc nulls last
limit 5
```
<DataTable data={pkg_100} wrapTitles=true subtotals=true totalRow=true groupType=section>
    <Column id=thumbnail contentType=html/>
    	<Column id=product_name wrap=true/>
    	<Column id=revenue title="Revenue (tỷ đồng)" fmt= '#,##0.00,,,'  contentType=colorscale/>
    	<Column id=platform_id/>
    	<Column id=price title="Giá (Nghìn đồng)"  fmt = "#,##0.00," />
    	<Column id=cleaned_brand title="Thương hiệu"/>
      
</DataTable>

<div class="bg-[#FFFFFF] w-[20px] h-[350px]"></div>

## Phân khúc giá từ 100k đến 200k
```sql pkg_200
select product_name,
       (revenue_202301 + revenue_202302 + revenue_202303 + revenue_202304 + revenue_202305 + revenue_202306 +
            revenue_202307 + revenue_202308 + revenue_202309 + revenue_202310 + revenue_202311 + revenue_202312) as revenue,
       concat('<img src="', url_thumbnail,'" width="300px" />') as thumbnail,
       platform_id,
       price,
       cleaned_brand 
from metricda.product
where price >= 100000 and price < 200000
        and cleaned_brand not in ('chưa biết', 'no brand')
order by (revenue_202301 + revenue_202302 + revenue_202303 + revenue_202304 + revenue_202305 + revenue_202306 +
            revenue_202307 + revenue_202308 + revenue_202309 + revenue_202310 + revenue_202311 + revenue_202312) desc nulls last
limit 5
```
<DataTable data={pkg_200} wrapTitles=true subtotals=true totalRow=true groupType=section>
    <Column id=thumbnail contentType=html/>
    	<Column id=product_name wrap=true/>
    	<Column id=revenue title="Revenue (tỷ đồng)" fmt= '#,##0.00,,,'  contentType=colorscale/>
    	<Column id=platform_id/>
    	<Column id=price title="Giá (Nghìn đồng)"  fmt = "#,##0.00," />
    	<Column id=cleaned_brand title="Thương hiệu"/>
      
</DataTable>


<div class="bg-[#FFFFFF] w-[20px] h-[350px]"></div>

## Phân khúc giá từ 200k đến 350k
```sql pkg_350
select product_name,
       (revenue_202301 + revenue_202302 + revenue_202303 + revenue_202304 + revenue_202305 + revenue_202306 +
            revenue_202307 + revenue_202308 + revenue_202309 + revenue_202310 + revenue_202311 + revenue_202312) as revenue,
       concat('<img src="', url_thumbnail,'" width="300px" />') as thumbnail,
       platform_id,
       price,
       cleaned_brand 
from metricda.product
where price >= 200000 and price < 350000
        and cleaned_brand not in ('chưa biết', 'no brand')
order by (revenue_202301 + revenue_202302 + revenue_202303 + revenue_202304 + revenue_202305 + revenue_202306 +
            revenue_202307 + revenue_202308 + revenue_202309 + revenue_202310 + revenue_202311 + revenue_202312) desc nulls last
limit 5
```
<DataTable data={pkg_350} wrapTitles=true subtotals=true totalRow=true groupType=section>
    <Column id=thumbnail contentType=html/>
    	<Column id=product_name wrap=true/>
    	<Column id=revenue title="Revenue (tỷ đồng)" fmt= '#,##0.00,,,'  contentType=colorscale/>
    	<Column id=platform_id/>
    	<Column id=price title="Giá (Nghìn đồng)"  fmt = "#,##0.00," />
    	<Column id=cleaned_brand title="Thương hiệu"/>
      
</DataTable>


<div class="bg-[#FFFFFF] w-[20px] h-[350px]"></div>

## Phân khúc giá trên 350k
```sql pkg_tren_350
select product_name,
       (revenue_202301 + revenue_202302 + revenue_202303 + revenue_202304 + revenue_202305 + revenue_202306 +
            revenue_202307 + revenue_202308 + revenue_202309 + revenue_202310 + revenue_202311 + revenue_202312) as revenue,
       concat('<img src="', url_thumbnail,'" width="300px" />') as thumbnail,
       platform_id,
       price,
       cleaned_brand 
from metricda.product
where price >= 350000
        and cleaned_brand not in ('chưa biết', 'no brand')
order by (revenue_202301 + revenue_202302 + revenue_202303 + revenue_202304 + revenue_202305 + revenue_202306 +
            revenue_202307 + revenue_202308 + revenue_202309 + revenue_202310 + revenue_202311 + revenue_202312) desc nulls last
limit 5
```
<DataTable data={pkg_tren_350} wrapTitles=true subtotals=true totalRow=true groupType=section>
    <Column id=thumbnail contentType=html/>
    	<Column id=product_name wrap=true/>
    	<Column id=revenue title="Revenue (tỷ đồng)" fmt= '#,##0.00,,,'  contentType=colorscale/>
    	<Column id=platform_id/>
    	<Column id=price title="Giá (Nghìn đồng)"  fmt = "#,##0.00," />
    	<Column id=cleaned_brand title="Thương hiệu"/>
      
</DataTable>