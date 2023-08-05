<template>
  <div class="panel">
    <input
      type="file"
      class="file"
      ref="fileInput"
      @change="handleFileUpload"
      accept=".xlsx, .xls"
    />
    <div class="button-upload" @click="openFileInput">Загрузить Excel</div>
    <div class="button-download" @click="downloadImages" >Скачать все</div>
    <img src="/loader.gif" alt="" class="loader" v-if="loader">
  </div>

  <div class="chart-wrap">
    <div
      class="chart-item"
      v-for="(item, index) in dataArray"
      :key="index"
      ref="divBlock"
    >
      <div class="main-stat">
        <div class="consum">{{ item[4].toFixed(2) }} л / 100км</div>
        <div class="tonne">{{ item[1].toFixed(0) }}т</div>
      </div>
      <div
        class="chart chart-rec"
        style="--progress: 100%; --color: #003049; --border-width: 20px"
      ></div>

      <div
        class="chart chart-mileage"
        :style="{
          '--progress': (item[3].toFixed(0) / item[2].toFixed(0)) * 100 + '%',
          '--color': '#3a86ff',
          '--border-width': '20px',
        }"
      >
        <div class="stat-rec-mileage">
          <span class="stat-mile-text">{{ item[3].toFixed(2) }} / </span>
          <span class="stat-rec-text"
            >{{ item[2] % 1 !== 0 ? item[2].toFixed(2) : item[2] }} км
          </span>
        </div>
      </div>

      <div
        class="chart chart-eek"
        :style="{
          '--progress': item[5].toFixed(2) + '%',
          '--color': '#8338ec',
          '--border-width': '20px',
        }"
      >
        <div class="stat-eek">
          <span class="stat-eek-text">{{ item[5].toFixed(2) }}%</span>
        </div>
      </div>

      <div
        class="chart chart-accomp"
        :style="{
          '--progress': item[5].toFixed(2) + '%',
          '--color': '#ff006e',
          '--border-width': '20px',
        }"
      >
        <div class="stat-accomp">
          <span class="stat-accomp-text">{{ item[6].toFixed(2) }}%</span>
        </div>
      </div>

      <div class="car-number">
        <div class="number-part letter">{{ item[0][0] }}</div>
        <div class="number-part digits">
          {{ item[0][1] + item[0][2] + item[0][3] }}
        </div>
        <div class="number-part letters">{{ item[0][4] + item[0][5] }}</div>
        <div class="number-part region">
          {{ item[0][6] + item[0][7] + item[0][8] }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import * as XLSX from "xlsx";
import JSZip from "jszip";
import * as htmlToImage from 'html-to-image';

export default {
  data() {
    return {
      dataArray: [],
      loader: false
    };
  },
  methods: {
    openFileInput() {
      this.$refs.fileInput.click();
    },
    handleFileUpload(event) {
      const file = event.target.files[0];
      const reader = new FileReader();

      reader.onload = (e) => {
        const data = new Uint8Array(e.target.result);
        const workbook = XLSX.read(data, { type: "array" });

        const firstSheetName = workbook.SheetNames[0];
        const worksheet = workbook.Sheets[firstSheetName];

        const jsonData = XLSX.utils.sheet_to_json(worksheet, { header: 1 });
        jsonData.splice(0, 1);

        this.dataArray = jsonData;
        console.log(jsonData);
      };

      reader.readAsArrayBuffer(file);
    },
    async downloadImages() {
      this.loader = true;
      const zip = new JSZip();

      const promises = [];

      for (let i = 0; i < this.$refs.divBlock.length; i++) {
        const divBlock = this.$refs.divBlock[i];

        const promise = htmlToImage.toPng(divBlock).then((dataUrl) => {
          const blob = this.dataURLtoBlob(dataUrl);
          return { filename: `image_${this.dataArray[i][0]}.png`, blob };
        });

        promises.push(promise);
      }

      Promise.all(promises).then((results) => {
        results.forEach((result) => {
          zip.file(result.filename, result.blob);
        });

        zip.generateAsync({ type: 'blob' }).then((content) => {
          const downloadLink = document.createElement('a');
          downloadLink.href = URL.createObjectURL(content);
          downloadLink.download = 'images.zip';
          downloadLink.click();
          this.loader = false;
        });
      });
    },
    dataURLtoBlob(dataURL) {
      const arr = dataURL.split(',');
      const mime = arr[0].match(/:(.*?);/)[1];
      const bstr = atob(arr[1]);
      let n = bstr.length;
      const u8arr = new Uint8Array(n);
      while (n--) {
        u8arr[n] = bstr.charCodeAt(n);
      }
      return new Blob([u8arr], { type: mime });
    },
  },
};
</script>

<style scoped lang="scss">
.file {
  display: none;
}

.chart-wrap {
  margin: 20px;
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-gap: 32px;
  min-height: 100vh;

  .chart-item {
    display: flex;
    justify-content: center;
    align-items: center;
    position: relative;
    width: 450px;
    height: 450px;
    background: #fff;

    .main-stat {
      position: absolute;
      z-index: 40;
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 8px;

      .consum {
        font-size: 16px;
        font-weight: 500;
        color: #fff;
        background: #ffb33a;
        padding: 5px 10px;
        border-radius: 32px;
      }

      .tonne {
        font-size: 20px;
        font-weight: 500;
      }
    }

    .chart-circle {
      position: absolute;
      left: 50%;
      top: 50%;
      transform: translate(-50%, -55%);
      z-index: 20;
      width: 160px;
      height: 160px;

      &::before {
        content: "";
        display: block;
        width: 130px;
        height: 130px;
        background-color: rgb(255, 255, 255);
        border-radius: 50%;
        position: absolute;
        left: 50%;
        top: 50%;
        transform: translate(-50%, -50%);
      }
    }

    .chart-mileage {
      @extend .chart-circle;

      height: 170px;
      width: 170px;
      z-index: 20;

      &::before {
        width: 130px;
        height: 130px;
      }
    }

    .chart-rec {
      @extend .chart-circle;

      height: 170px;
      width: 170px;
      z-index: 15;

      &::before {
        width: 130px;
        height: 130px;
      }
    }

    .chart-eek {
      @extend .chart-circle;
      transform: translate(-50%, -55%) scale(-1, 1);
      width: 260px;
      height: 260px;
      z-index: 10;

      &::before {
        width: 220px;
        height: 220px;
      }
    }

    .chart-accomp {
      @extend .chart-circle;
      width: 350px;
      height: 350px;
      z-index: 5;

      &::before {
        width: 310px;
        height: 310px;
      }
    }
  }
}

.chart {
  border-radius: 50%;
  background-image: conic-gradient(var(--color) var(--progress), transparent 0);
  position: relative;
}

/* stats */

.stat-rec-mileage {
  position: relative;
  border-radius: 8px;
  padding: 6px 8px;
  margin: 0 auto;
  width: max-content;
  text-align: center;
  background-image: linear-gradient(90deg, #3a86ff, #003049 100%);
  background-origin: border-box;
  box-shadow: inset 0 1000px white;
  border: 2px solid transparent;
  font-weight: 500;

  .stat-mile-text {
    color: #3a86ff;
  }

  .stat-rec-text {
    color: #003049;
  }
}

.stat-eek {
  min-width: 80px;
  transform: scale(-1, 1);
  position: relative;
  border-radius: 8px;
  padding: 6px 8px;
  margin: -5px auto;
  width: max-content;
  text-align: center;
  background-image: linear-gradient(90deg, #8338ec, #8338ec 100%);
  background-origin: border-box;
  box-shadow: inset 0 1000px white;
  border: 2px solid transparent;
  font-weight: 500;

  .stat-eek-text {
    color: #8338ec;
  }
}

.stat-accomp {
  min-width: 80px;
  position: relative;
  border-radius: 8px;
  padding: 6px 8px;
  margin: -5px auto;
  width: max-content;
  text-align: center;
  background-image: linear-gradient(90deg, #ff006e, #ff006e 100%);
  background-origin: border-box;
  box-shadow: inset 0 1000px white;
  border: 2px solid transparent;
  font-weight: 500;

  .stat-accomp-text {
    color: #ff006e;
  }
}

/* car number */

.car-number {
  position: absolute;
  z-index: 99;
  bottom: 0px;
  display: flex;
  font-size: 24px;
  background-color: #fff;
  border-radius: 8px;
  border: 2px solid #000;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);

  .number-part {
    padding: 0px;
    font-weight: bold;
  }

  .letter {
    color: #000;
    border-radius: 4px 0 0 4px;
    margin-right: 2px;
    padding: 6px 0 6px 6px;
  }

  .letters {
    color: #000;
    padding: 6px 0;
  }

  .digits {
    color: #000;
    border-radius: 0 4px 4px 0;
    margin: 0 2px 0 0;
    padding: 6px 0;
  }

  .region {
    padding: 6px;
    margin-left: 12px;
    border-left: 1px solid #000;
  }
}

/* panel */

.panel {
  display: flex;
  align-items: center;

  .button-upload {
    margin: 16px;
    border-radius: 10px;
    max-width: 200px;
    display: flex;
    align-items: center;
    justify-content: center;
    background: #44c13c;
    color: #fff;
    padding: 10px 20px;
    cursor: pointer;
  }

  .button-download {
    border-radius: 10px;
    max-width: 200px;
    display: flex;
    align-items: center;
    justify-content: center;
    background: #4040f2;
    color: #fff;
    padding: 10px 20px;
    cursor: pointer;
  }
}

.loader {
  width: 25px;
  margin-left: 16px;
}
</style>
