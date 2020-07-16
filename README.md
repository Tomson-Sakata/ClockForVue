# ClockForVue
It's just a component that displays a clock for Vue.

# Requirement
* Vue 2.6.11

# Screen shot
![画面](https://github.com/Tomson-Sakata/ClockForVue/blob/images/screenshot_1.jpg)

# Usage
1.copy src/components/Clock.vue to your project.

2.Embedded in the components of your project.

    <template>
        <div>
            <Clock type="digital_normal" />
            <br/>
            <div style="border:1px solid; border-color:gray; border-radius:8px; display:inline-block">
                <Clock type="digital_normal" ampmPosition="before" color="#00897B" />
            </div><br/>
            <br/>
            <Clock type="digital_mini" :isShowMilitaryTime="true" :isShowAmPm="false" />
            <br/>
            <Clock type="analog_icon" />
            <br/>
            <br/>
            <div style="width:50%;"><Clock type="analog_normal" /></div>
        </div>
    </template>

    <script>
        import Clock from '@/components/Clock'
        export default {
            name: 'HelloWorld',
            components: {
                Clock
            }
        }
    </script>

# API

* Properties

|name|type|default|desctiption|
|:---|:---|:---|:---|
|type|String|"digital_normal"|Specify the type of clock to be displayed from one of the following.<br/>"digital_normal","digital_mini","analog_normal","analog_icon"|
|isShowMilitaryTime|Boolean|false|24-hour display or not(*1)|
|isShowAmPm|Boolean|true|"AM"/"PM" are displayed or not(*1)|
|ampmPosition|String|"after"|Position to display "AM"/"PM"(*1)(*2)|
|color|String|"#000000"|Text color(*1)|
|bodyLineColor|Object|{r:0, g:0, b:0, a:1}|Frame color of the watch body(*3)|
|bodyFillColor|Object|null|Color of the watch body(*3)|
|dialColor|Object|{r:0, g:0, b:0, a:1}|Color of the clock face(*3)|
|handHourColor|Object|{r:0, g:0, b:0, a:1}|Color of the watch's long hand(*3)|
|handMinuteColor|Object|{r:0, g:0, b:0, a:1}|Color of the clock's short hand(*3)|
|handSecondColor|Object|{r:255, g:0, b:0, a:1}|Color of the clock's second hand(*3)|

(*1) ... Valid only when type="digital_normal" or "digital_mini".

(*2) ... Valid only when isShowAmPm=true.

(*3) ... Valid only when type="analog_normal" or "analog_icon".
