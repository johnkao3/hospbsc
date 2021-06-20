<template>
  <q-page padding>
    <div>
      檔案來源
      <a
        href="https://www.nhi.gov.tw/Content_List.aspx?n=07FEBAA0B8C34D90&topn=D39E2B72B0BDFA15"
        target="_blank"
      >
        衛生福利部中央健康保險署-特約醫事機構代碼網路查詢服務(另開視窗)
      </a>
    </div>
    <q-separator />
    <p class="text-body1">用於快速轉出所需的縣市「健保特約醫療院所名冊」並維持相同規範轉出TXT。</p>
    <q-card>
      <q-card-section>
        <q-file
          v-model="fileModel"
          label="健保特約醫療院所名冊"
          accept=".txt"
          @input="loadFile"
        />
      </q-card-section>
    </q-card>
    <div class="text-h6">總筆數: {{ newData.length }}，已排除以下條件:終止合約或歇業日期</div>
    <q-card>
      <q-card-section>
        <div class="text-h6">轉出條件</div>
      </q-card-section>

      <q-card-section class="q-pt-none">
        <div class="q-gutter-sm">
          <template v-for="(value, name) in city">
            <q-checkbox v-model="cityModel" :val="value" :label="name" :key="value" />
          </template>
        </div>
      </q-card-section>
      <q-card-actions>
        <q-btn
          color="primary"
          label="轉出並下載"
          @click="exportData"
          :disable="fileModel === null"
        />
      </q-card-actions>
    </q-card>
    <!-- {{ newData }} -->
  </q-page>
</template>

<script>
import { exportFile } from 'quasar'
const city = {
  臺北市: 'A',
  臺中市: 'B',
  基隆市: 'C',
  臺南市: 'D',
  高雄市: 'E',
  新北市: 'F',
  宜蘭縣: 'G',
  桃園縣: 'H',
  新竹縣: 'J',
  苗栗縣: 'K',
  臺中縣: 'L',
  南投縣: 'M',
  彰化縣: 'N',
  雲林縣: 'P',
  嘉義縣: 'Q',
  臺南縣: 'R',
  高雄縣: 'S',
  屏東縣: 'T',
  花蓮縣: 'U',
  臺東縣: 'V',
  澎湖縣: 'X',
  金門縣: 'W',
  連江縣: 'Z',
  嘉義市: 'I',
  新竹市: 'O'
}

export default {
  name: 'PageIndex',
  data () {
    return {
      cityModel: [],
      fileModel: null,
      newData: [],
      city: city
    }
  },
  methods: {
    loadFile (inputFile) {
      this.$q.loading.show()

      const input = inputFile
      const reader = new FileReader()
      const $this = this
      reader.onload = function () {
        const fileData = reader.result
        const arr = []
        const textArray = fileData.split('\n')
        for (let i = 1; i < textArray.length; ++i) {
          const elementReplace = textArray[i].replace(/['"]|\r+/g, '')
          if (elementReplace.length !== 0) {
            const obj = {}
            const hospital = elementReplace.split(',')
            if (hospital[10] !== '2' && hospital[9] === '') {
              obj.partition = hospital[0]
              obj.code = hospital[1]
              obj.name = hospital[2]
              obj.address = hospital[3]
              obj.phoneCode = hospital[4]
              obj.phone = hospital[5]
              obj.category = hospital[6]
              obj.type = hospital[7]
              obj.class = hospital[8]
              obj.closingDate = hospital[9]
              obj.status = hospital[10]
              // ex
              obj.cityCode = city[hospital[3].slice(0, 3)]
              arr.push(obj)
            }
          }
        }
        $this.$data.newData = arr
        $this.$q.loading.hide()
      }
      reader.readAsText(input, 'utf-16le')
    },
    exportData () {
      this.$q.loading.show()
      let regexp = ''
      this.$data.cityModel.forEach((element, index) => {
        if (index === (this.$data.cityModel.length - 1)) {
          regexp += `${element}`
        } else {
          regexp += `${element}|`
        }
      })
      const newRegExp = new RegExp(regexp)
      const newData = this.$data.newData
      let exportData = '分區別,醫事機構代碼,醫事機構名稱,機構地址,電話區域號碼 ,電話號碼,特約類別,型態別,醫事機構種類,終止合約或歇業日期,開業狀況\r\n'
      for (let i = 0, len = newData.length; i < len; i++) {
        if (newRegExp.test(newData[i].cityCode) === true) {
          const obj = newData[i]
          exportData += `"${obj.partition}","${obj.code}","${obj.name}","${obj.address}","${obj.phoneCode}","${obj.phone}","${obj.category}","${obj.type}","${obj.class}","${obj.closingDate}","${obj.status}"\r\n`
        }
      }
      let charCode = ''
      const byteArray = []
      byteArray.push(255, 254)
      for (let i = 0, len = exportData.length; i < len; i++) {
        charCode = exportData.charCodeAt(i)
        byteArray.push(charCode & 0xff)
        byteArray.push(charCode / 256 >>> 0)
      }
      this.$q.loading.hide()
      const fileName = `${Date.now()}_hospbsc.txt`
      exportFile(fileName, new Uint8Array(byteArray), 'text/plain;charset=UTF-16LE;')
    }
  }
}
</script>
