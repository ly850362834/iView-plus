<template>
    <TransferPlus
            :data="data3"
            :target-keys="targetKeys3"
            :list-style="listStyle"
            :render-format="render3"
            :operations="['To left','To right']"
            filterable
            :leftPage="leftPage"
            :splitPage="0"
            @leftNextPage="leftNextPage"
            @on-change="handleChange3">
        <div :style="{float: 'right', margin: '5px'}">
            <Button type="ghost" size="small" @click="reloadMockData">Refresh</Button>
        </div>
    </TransferPlus>
</template>
<script>
    export default {
        data () {
            return {
                data3: this.getMockData(),
                targetKeys3: this.getTargetKeys(),
                listStyle: {
                    width: '250px',
                    height: '300px'
                },
                leftPage:1
            };
        },
        methods: {
            leftNextPage(){
            },
            getMockData () {
                let mockData = [];
                for (let i = 1; i <= 10; i++) {
                    mockData.push({
                        key: i.toString(),
                        label: 'Content ' + i,
                        description: 'The desc of content  ' + i,
                        disabled: Math.random() * 3 < 1
                    });
                }
                return mockData;
            },
            getTargetKeys () {
                return this.getMockData()
                    .filter(() => Math.random() * 2 > 1)
                    .map(item => item.key);
            },
            handleChange3 (newTargetKeys) {
                this.targetKeys3 = newTargetKeys;
            },
            render3 (item) {
                return item.label + ' - ' + item.description;
            },
            reloadMockData () {
                this.data3 = this.getMockData();
                this.targetKeys3 = this.getTargetKeys();
            }
        }
    }
</script>
