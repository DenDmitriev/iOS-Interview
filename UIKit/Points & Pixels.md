# Points & Pixels
Пиксели и точки - это еденицы измерения экрана.

## Пиксель
Это одна группа цветных субпикселей (обычно красны, два зеленых и синий) на экране. Включив и выключив их с разной интенсивностью, можно создать любой цвет и яркость.
1 пиксель - это всегда 1 пиксель и является самой маленькой частью экрана, которая может отображать цвета. 

## DPI (точки на дюйм)
это число, которое измеряет, сколько пикселей содержится в одном дюйме экранного пространства. 
На iPhone 1 DPI было равно 163. Это значит что в 1 дюйме было 163 пикселя. А вот iPhone 4, 5 и 6 имели DPI равный 326 пикселям на дюйм. То есть DPI увеличился в два раза.


## Точка
Это абстрактная единица равная 1 ÷ 163 части дюйма экрана, имеющей величину только при упоминании размера DPI экрана. 
1 точка всегда 1 точка и является абстрактной единицей, имеющей значение только при упоминании по отношению к другим точкам. 
Можно сформулировать другое определение: Точка - это колличество пикселей в 1 ÷ 163 части экрана устройства.

## Преобразование между пикселями и точками
Точки отличаются от пикселей, потому что они меняют размер в зависимости от DPI экрана:
- На iPhone 1 точка равна 1 пикселю, так как разрешение составляет 163 точек на дюйм. Расчет:
DPI = 163 пикселей на дюйм
point = DPI ÷ 163 = 163 ÷ 163 = 1

- iPhone 4, 5 и 6 имели DPI равный 326 пикселям на дюйм. 
DPI = 326 пикселей на дюйм
point = DPI ÷ 163 = 326 ÷ 163 = 2

## Таблица DPI (PPI)
<table>
  <tbody>
      <tr>
          <td class="has-text-align-center" data-align="center" colspan="4">
              <strong>Display properties of different types of iPhones</strong>
          </td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">
              <strong>iPhone Model</strong>
          </td>
          <td class="has-text-align-center" data-align="center">
              <strong>Resolution in pixels</strong>
              <br>
              <strong>(Width x Height)</strong>
          </td>
          <td class="has-text-align-center" data-align="center">
              <strong>Resolution in points</strong>
              <br>
              <strong>(Width x Height)</strong>
          </td>
          <td class="has-text-align-center" data-align="center">
              <strong>PPI</strong>
          </td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone</td>
          <td class="has-text-align-center" data-align="center" rowspan="3">320 x 480</td>
          <td class="has-text-align-center" data-align="center" rowspan="5">320 x 480</td>
          <td class="has-text-align-center" data-align="center" rowspan="3">163</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone 3G</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone 3GS</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone 4</td>
          <td class="has-text-align-center" data-align="center" rowspan="2">640 x 960</td>
          <td class="has-text-align-center" data-align="center" rowspan="12">326</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone 4S</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone 5</td>
          <td class="has-text-align-center" data-align="center" rowspan="4">640 x 1136</td>
          <td class="has-text-align-center" data-align="center" rowspan="4">320 x 568</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone 5S</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone 5C</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone SE {1st gen}</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone 6</td>
          <td class="has-text-align-center" data-align="center" rowspan="6">750 x 1334</td>
          <td class="has-text-align-center" data-align="center" rowspan="6">375 x 667</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone 6S</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone 7</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone 8</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone SE {2nd gen}</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone {3rd gen}</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone 6 Plus</td>
          <td class="has-text-align-center" data-align="center" rowspan="4">1080 x 1920</td>
          <td class="has-text-align-center" data-align="center" rowspan="4">414 x 736</td>
          <td class="has-text-align-center" data-align="center" rowspan="4">401</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone 6S Plus</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone 7 Plus</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone 8 Plus</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone X</td>
          <td class="has-text-align-center" data-align="center" rowspan="3">1125 x 2436</td>
          <td class="has-text-align-center" data-align="center" rowspan="3">375 x 812</td>
          <td class="has-text-align-center" data-align="center" rowspan="5">
              <br>
              458
          </td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone XS</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone 11 Pro</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone XS Max</td>
          <td class="has-text-align-center" data-align="center" rowspan="2">1242 x 2688</td>
          <td class="has-text-align-center" data-align="center" rowspan="4">414 x 896</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone 11 Pro Max</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone XR</td>
          <td class="has-text-align-center" data-align="center" rowspan="2">828 x 1792</td>
          <td class="has-text-align-center" data-align="center" rowspan="2">326</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone 11</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone 12 Pro</td>
          <td class="has-text-align-center" data-align="center" rowspan="5">1170 x 2532</td>
          <td class="has-text-align-center" data-align="center" rowspan="5">390 x 844</td>
          <td class="has-text-align-center" data-align="center" rowspan="5">460</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone 12</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone 13 Pro</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone 13</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone 14</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone 12 Pro Max</td>
          <td class="has-text-align-center" data-align="center" rowspan="3">1284 x 2778</td>
          <td class="has-text-align-center" data-align="center" rowspan="3">428 x 926</td>
          <td class="has-text-align-center" data-align="center" rowspan="3">458</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone 13 Pro Max</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone 14 Plus</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone 12 mini</td>
          <td class="has-text-align-center" data-align="center" rowspan="2">1080 x 2340</td>
          <td class="has-text-align-center" data-align="center" rowspan="2">375 x 812</td>
          <td class="has-text-align-center" data-align="center" rowspan="2">476</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone 13 mini</td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone 14 Pro</td>
          <td class="has-text-align-center" data-align="center">1179 x 2556</td>
          <td class="has-text-align-center" data-align="center">393 x 852</td>
          <td class="has-text-align-center" data-align="center" rowspan="2">
              <br>
              460
          </td>
      </tr>
      <tr>
          <td class="has-text-align-center" data-align="center">iPhone 14 Pro Max</td>
          <td class="has-text-align-center" data-align="center">
              <span style="font-size: 15px;">1290 x 2796 </span>
          </td>
          <td class="has-text-align-center" data-align="center">430 x 932</td>
      </tr>
  </tbody>
</table>

## Источники
- [Mobile design 101: pixels, points and resolutions](https://medium.com/@fluidui/mobile-design-101-pixels-points-and-resolutions-f60413035243)
