<template>
  <div class="index-page">
    <a-input-search
      v-model:value="searchText"
      placeholder="请输入搜索关键词"
      enter-button="搜索"
      size="large"
      @search="onSearch"
    />
    <MyDivider />
    <a-tabs v-model:activeKey="activeKey" @change="onTabChange">
      <a-tab-pane key="post" tab="文章">
        <PostList :post-list="postList" />
      </a-tab-pane>
      <a-tab-pane key="picture" tab="图片">
        <PictureList :picture-list="pictureList" />
      </a-tab-pane>
      <a-tab-pane key="user" tab="用户">
        <UserList :user-list="userList" />
      </a-tab-pane>
      <a-tab-pane key="video" tab="视频">
        <VideoList :video-list="videoList"></VideoList>
      </a-tab-pane>
    </a-tabs>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref, watchEffect } from "vue";
import PostList from "@/components/PostList.vue";
import PictureList from "@/components/PictureList.vue";
import UserList from "@/components/UserList.vue";
import MyDivider from "@/components/MyDivider.vue";
import { useRoute, useRouter } from "vue-router";
import myAxios from "@/plugins/myAxios";
import { message } from "ant-design-vue";
import VideoList from "@/components/VideoList.vue";

const postList = ref([]);

const userList = ref([]);

const pictureList = ref([]);

const videoList = ref([]);

const route = useRoute();
const router = useRouter();
const activeKey = route.params.category;

const initSearchParams = {
  text: "",
  pageSize: 50,
  pageNum: 1,
};

let tabKey = route.params.category || "post";
const searchText = ref(route.query.text || "");

/**
 * 加载聚合数据（所有数据）
 * @param params
 */
const loadAllData = (params: any) => {
  const query = {
    ...params,
    searchText: searchText,
  };
  if (params.text == null) {
    message.info("开启愉快的搜索吧！");
  } else {
    myAxios.post("search/all", query).then((res: any) => {
      postList.value = res.postList;
      userList.value = res.userList;
      pictureList.value = res.pictureList;
      videoList.value = res.videoList;
    });
  }
};

/**
 * 加载单类数据
 * @param params
 * @param type
 */
const loadData = (params: any) => {
  if (!tabKey) {
    message.error("类别为空");
    return;
  }
  const query = {
    ...params,
    type: tabKey,
    searchText: params.text,
  };
  if (params.text != null) {
    message.loading("搜索中", 1);
    myAxios.post("search/all", query).then((res: any) => {
      if (tabKey === "post") {
        postList.value = res.dataList;
      } else if (tabKey === "user") {
        userList.value = res.dataList;
      } else if (tabKey === "picture") {
        pictureList.value = res.dataList;
      } else if (tabKey === "video") {
        videoList.value = res.dataList;
      }
    });
  } else {
    message.info("输入内容，开启愉快的搜索吧！");
  }
};

const searchParams = ref(initSearchParams);

// 从url中加载数据
onMounted(() => {
  searchParams.value = {
    pageSize: route.query.pageSize,
    pageNum: route.query.pageNum,
    text: route.query.text,
  } as any;
  loadData(searchParams.value);
});

const onSearch = (value: string) => {
  if (value != searchParams.value.text) {
    router.push({
      path: `/${tabKey}`,
      query: {
        ...searchParams.value,
        text: value,
      },
    });
    searchParams.value = {
      ...initSearchParams,
      text: value,
    } as any;
    loadData(searchParams.value);
  } else {
    return;
  }
};

const onTabChange = (key: string) => {
  tabKey = key;
  searchParams.value = {
    ...initSearchParams,
    text: searchText.value,
  } as any;
  router.push({
    path: `/${tabKey}`,
    query: searchParams.value,
  });
  loadData(searchParams.value);
};
</script>
