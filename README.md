# ClockForVue
It's just a component that displays a clock for Vue.

# Requirement
* Vue 2.6.11

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

    Properties

        type ... Type of clock to be displayed. ("digital_normal" , "digital_mini" , "analog_normal" , "analog_icon") (default: "digital_normal")
        isShowMilitaryTime ... 24-hour display or not. (default: false)
        isShowAmPm ... "AM"/"PM" are displayed or not. (default: true)
        ampmPosition ... Position to display "AM"/"PM" ("before" , "after") (default: "after")
        color ... Text color (default: "black")

        *Currently, only digital clocks are supported for each display property.
