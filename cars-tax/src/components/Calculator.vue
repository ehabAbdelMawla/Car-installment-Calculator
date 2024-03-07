<template>
  <Header />

  <div class="inputs-container wow fadeIn" id="capture">
    <article :class="{ result: true, wow: true, fadeInDown: true }">
      <div class="form-group">
        <label for="car-price">سعر السيارة</label>
        <input id="car-price" type="number" placeholder="0" v-model="price" />
      </div>
      <div class="form-group">
        <label for="car-down-payment-value">قيمة المقدم</label>
        <input
          id="car-down-payment-value"
          :type="isAndroid ? 'tel' : 'number'"
          placeholder="0"
          v-model="downpaymentValue"
          @input="handleChangeDownpaymentValue($event.target.value)"
          :readonly="price == 0"
        />
      </div>
      <div class="form-group">
        <label for="car-down-payment-percent">نسبة المقدم</label>
        <input
          id="car-down-payment-percent"
          :type="isAndroid ? 'tel' : 'number'"
          placeholder="0"
          v-model="downpaymentPercent"
          @input="handleChangeDownpaymentPercentage($event.target.value)"
          :readonly="price == 0"
        />
      </div>
      <div class="form-group" data-html2canvas-ignore="true">
        <label for="car-down-payment-percent">نسبة الفائدة</label>
        <input
          id="car-down-payment-percent"
          type="number"
          placeholder="0"
          v-model="additions"
          :readonly="price == 0"
        />
      </div>
    </article>
    <div
      :class="{
        result: true,
        wow: true,
        fadeInDown: true,
      }"
    >
      <div>
        <p>قيمة التمويل</p>
        <p>{{ remain }}</p>
      </div>
      <div>
        <p>مصاريف ادارية</p>
        <p>{{ masreefEdaria }}</p>
      </div>
      <div>
        <p>تأمين</p>
        <p>{{ taminat }}</p>
      </div>
    </div>
    <div
      :class="{
        result: true,
        wow: true,
        fadeInDown: true,
      }"
      id="breakAfter"
    >
      <div v-for="num in 7" :key="num">
        <p>{{ getYearsSuffix(num) }}</p>
        <p>{{ taxPerYear(num) }}</p>
      </div>
    </div>
  </div>
  <button class="printBtn" @click="printToPdf" data-html2canvas-ignore="true">
    <i class="fa fa-file-pdf" />
  </button>
  <button
    class="sreenBtn"
    @click="takeScreenShot"
    data-html2canvas-ignore="true"
  >
    <i class="fa fa-camera-retro" />
  </button>
</template>

<script>
import "../styles/Calculator.css";
import html2canvas from "html2canvas";
import html2pdf from "html2pdf.js";
import swal from "sweetalert";
import { Filesystem, Directory } from "@capacitor/filesystem";
import Header from "./Header.vue";
import { App } from "@capacitor/app";
export default {
  name: "Calculator",
  components: {
    Header,
  },
  data() {
    return {
      price: "",
      downpaymentValue: "",
      downpaymentPercent: "",
      additions: "",
      percentWithoutFormat: "",
      isAndroid: /Android/i.test(navigator.userAgent),
    };
  },
  created() {
    document.addEventListener("backbutton", async function() {
      App.exitApp();
    });
  },
  methods: {
    handleChangeDownpaymentValue(value) {
      if (this.price > 0) {
        this.percentWithoutFormat = (value / this.price) * 100;
        this.downpaymentPercent = this.shortFormat(this.percentWithoutFormat);
      } else {
        this.downpaymentPercent = 0;
      }
      return value;
    },
    handleChangeDownpaymentPercentage(value) {
      this.percentWithoutFormat = value;
      if (this.price > 0) {
        this.downpaymentValue = this.shortFormat(
          (this.percentWithoutFormat / 100) * this.price
        );
      } else {
        this.downpaymentValue = 0;
      }
      return value;
    },
    shortFormat(number) {
      return Number(number).toFixed(2);
    },
    getYearsSuffix(num) {
      return num == 1 ? "سنة" : num == 2 ? "سنتين" : num + " سنين";
    },
    getFileName(message) {
      return swal(message, {
        content: "input",
      });
    },
    async takeScreenShot() {
      if (this.price > 0) {
        const fileName = await this.getFileName("Type Image Name:");

        window.scrollTo(0, 0);
        html2canvas(document.getElementById("capture")).then(async (canvas) => {
          if (this.isAndroid) {
            // .... Android ....
            await Filesystem.writeFile({
              path: fileName ? `${fileName}.png` : `${Date.now()}.png`,
              data: canvas.toDataURL(),
              directory: Directory.Documents,
            });
          } else {
            //    .... Web ....
            const aTag = document.createElement("a");
            aTag.setAttribute("href", canvas.toDataURL());
            aTag.setAttribute("download", fileName ? fileName : "image.png");
            aTag.click();
          }
          swal("Image Saved.", {
            icon: "success",
          });
        });
      }
    },
    async printToPdf() {
      if (this.price > 0) {
        let fileName = await this.getFileName("Type PDF Name:");
        fileName = fileName ? fileName + ".pdf" : "file.pdf";
        const element = document.getElementById("capture");
        const options = {
          margin: 1.5,
          html2canvas: { scale: 2 },
          filename: fileName,
          jsPDF: { unit: "in", format: "letter", orientation: "portrait" },
        };

        if (this.isAndroid) {
          const pdf = await html2pdf()
            .from(element)
            .set(options)
            .toPdf()
            .outputPdf();

          await Filesystem.writeFile({
            path: fileName,
            data: btoa(pdf),
            directory: Directory.Documents,
          });
          swal("PDF Saved.", {
            icon: "success",
          });
        } else {
          html2pdf()
            .from(element)
            .set(options)
            .toPdf()
            .outputPdf()
            .save();
        }
      }
    },
  },
  watch: {
    price(newValue) {
      if (newValue > 0) {
        if (this.downpaymentValue > 0) {
          this.percentWithoutFormat = (this.downpaymentValue / newValue) * 100;
          this.downpaymentPercent = this.shortFormat(this.percentWithoutFormat);
        } else if (this.downpaymentPercent > 0) {
          this.downpaymentValue = this.shortFormat(
            (this.downpaymentPercent / 100) * this.price
          );
        }
      }
    },
  },
  computed: {
    remain() {
      return Math.ceil(this.shortFormat(this.price - this.downpaymentValue));
    },
    masreefEdaria() {
      return Math.ceil(this.shortFormat(this.remain * 0.0249));
    },
    taminat() {
      return Math.ceil(this.shortFormat(this.price * 0.025));
    },
    taxPerYear() {
      return (numOfYear) => {
        const additions = Number(this.additions);
        const remain = Number(this.remain);
        return Math.ceil(
          this.shortFormat(
            (((numOfYear * additions) / 100) * remain + remain) /
              (numOfYear * 12)
          )
        );
      };
    },
  },
};
</script>
