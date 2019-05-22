<template>
    <div id="app">
        <el-container class="container">
            <el-aside width="200px">
                <my-header></my-header>
            </el-aside>
            <!-- <el-header>
               
            </el-header> -->
            <el-main class="container">
                <router-view></router-view>
            </el-main>
        </el-container>
    </div>
</template>

<script>
import MyHeader from './components/MyHeader'
export default {
    name: "App",
    components: {
        MyHeader
    }
};
</script>

