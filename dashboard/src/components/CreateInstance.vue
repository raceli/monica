<template>
    <Modal v-bind="$attrs" v-on="$listeners" :title="info && info.Path">
        <Steps :current="currentStep" size="small" :status="status">
            <Step title="解析请求"></Step>
            <Step title="创建目录"></Step>
            <Step title="写入文件"></Step>
            <Step title="执行go mod init"></Step>
            <Step title="执行go build"></Step>
            <Step title="启动实例"></Step>
            <Step title="完成"></Step>
        </Steps>
        <div>
            <pre>{{log}}</pre>
        </div>
        <div slot="footer">
            <Checkbox v-model="clearDir">安装前清空目录</Checkbox>
            <Button type="primary" @click="start" :loading="status=='process'">开始</Button>
        </div>
    </Modal>
</template>

<script>
let eventSource = null;
export default {
    name: "CreateInstance",
    props: {
        info: Object
    },
    methods: {
        start() {
            this.status = "process";
            eventSource = new EventSource(
                "/api/create?info=" +
                    JSON.stringify(this.info) +
                    (this.clearDir ? "&clear=true" : "")
            );
            eventSource.onopen = () => (this.log = "");
            eventSource.onmessage = evt => {
                this.log += evt.data + "\n";
                if (evt.data == "success") {
                    this.status = "finish";
                    eventSource.close();
                }
            };
            eventSource.addEventListener("exception", evt => {
                this.log += evt.data + "\n";
                this.status = "error";
                eventSource.close();
            });
            eventSource.addEventListener("step", evt => {
                let [step, msg] = evt.data.split(":");
                this.currentStep = step | 0;
                this.log += msg + "\n";
            });
        },
    },

    data() {
        return { clearDir: true, currentStep: 0, log: "", status: "wait" };
    }
};
</script>

<style scoped>
</style>