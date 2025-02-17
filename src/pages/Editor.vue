<template>
  <el-container class="editor-container">
    <el-aside width="300px">
      <attribute-tree :options="attribute" :values="example" @update="handleUpdateExample" />
    </el-aside>
    <el-container>
      <div :class="{ preivew: fullscreen }" :style="wallpaperStyles">
        <wallpaper
          :options="example"
          :width="screenWidth"
          :height="screenHeight"
          @on-success="canvas = $event"
        />
      </div>
      <div class="tools-container">
        <el-tooltip effect="dark" content="Story" placement="top">
          <el-button type="primary" icon="el-icon-present" circle @click="handleHelp"></el-button>
        </el-tooltip>
        <el-tooltip effect="dark" content="Preveiw" placement="top">
          <el-button type="primary" icon="el-icon-view" circle @click="handlePreview"></el-button>
        </el-tooltip>
        <el-popover placement="top" trigger="hover">
          <el-button icon="el-icon-files" type="primary" @click="handleDownloadFile" size="small"
            >File</el-button
          >
          <el-button
            icon="el-icon-picture-outline"
            type="primary"
            @click="handleDownloadImage"
            size="small"
            >Image</el-button
          >
          <el-button slot="reference" type="primary" icon="el-icon-download" circle></el-button>
        </el-popover>
      </div>
    </el-container>
  </el-container>
</template>

<script>
import { Message } from "element-ui";
import Wallpaper from "../components/Wallpaper.vue";
import AttributeTree from "../components/AttributeTree.vue";
import { getAttributeOptions } from "../utils/attribute";
import { downloadImage, downloadFile } from "../utils/file";
import { set, deepCopy } from "../utils/object";
import { useWindowSize } from "../mixins/useWindowSize";
import { useFullscreen } from "../mixins/useFullscreen";
import { color } from "../data/examples";

export default {
  components: {
    Wallpaper,
    AttributeTree,
  },
  name: "editor",
  data() {
    const example = localStorage.getItem("cd-example");
    return {
      example: example ? JSON.parse(example) : color,
      screenWidth: screen.width,
      screenHeight: screen.height,
    };
  },
  mixins: [useWindowSize(), useFullscreen()],
  computed: {
    wallpaperStyles() {
      const { transformed: t } = this;
      return this.fullscreen
        ? {}
        : {
            transform: `translate(${t.translateX}px, ${t.translateY}px) scale(${t.scale}, ${t.scale})`,
            transformOrigin: "left top",
          };
    },
    attribute() {
      return getAttributeOptions(this.example);
    },
    transformed() {
      const { screenWidth, screenHeight } = this;
      const padding = 50;
      const bottomToolsHeight = 25;
      const mainHeight = this.windowHeight - 61 - bottomToolsHeight;
      const mainWidth = this.windowWidth - 300;
      const width = mainWidth - padding * 2;
      const height = mainHeight - padding * 2;
      const sh = height / screenHeight;
      const sw = width / screenWidth;
      const scale = Math.min(sh, sw);
      const translateX = (mainWidth - screenWidth * scale) / 2;
      const translateY = (mainHeight - screenHeight * scale) / 2;
      return {
        scale,
        translateX,
        translateY,
      };
    },
  },
  methods: {
    handleUpdateExample({ key, value }) {
      // 每次更新的时候都深度拷贝一份
      // 这样 watcher 里面的 newData 和 oldData 就会不一样了
      this.example = deepCopy(this.example);
      set(this.example, key, value);
    },
    handleDownloadImage() {
      try {
        downloadImage(this.canvas, "wallpaper");
      } catch (e) {
        Message.info({
          message:
            "The online image can't be downloaded, You can take a screenshot in preview mode or upload local image!",
          duration: 5000,
        });
        console.log(e);
      }
    },
    handleDownloadFile() {
      downloadFile(this.example, "index");
    },
    handleHelp() {
      this.$router.push("/story");
    },
    async handlePreview() {
      try {
        await document.documentElement.requestFullscreen();
      } catch {
        Message.success("Failed to enter full screen");
      }
    },
  },
};
</script>

<style scoped>
.editor-container {
  height: calc(100vh - 61px);
}

.el-aside {
  background-color: #ffffff;
}

.el-container {
  background-color: #e5e5e5;
  overflow: hidden;
  padding: 0px;
}

.tools-container {
  position: fixed;
  bottom: 25px;
  right: 30px;
  z-index: 900;
}

.tools-container .el-button {
  margin: 0 6px;
}

.preivew {
  position: fixed;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  /** slider 的 z-index 是 1001 */
  z-index: 1005;
}
</style>