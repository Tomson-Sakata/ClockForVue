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
