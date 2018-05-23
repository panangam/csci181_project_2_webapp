<template>
  <q-page padding class="row justify-center" style="padding: 15px;">
  <div style="width: 800px; max-width: 90vw;">
    <label>
      CT
      <q-input
        v-model="ct_text"
        type="textarea"
        rows=3
        placeholder="Paste CT from Schaefer's file here"
      />
    </label>
    <q-btn label="update ct" color="primary" @click="updateCT()"/>
    
    <div class="row">
      <div class="ptBox col-2">
        <div class="row" style="padding: 15px 0">
          <q-input
            v-for="i in row_nums"
            v-bind:key="i"
            class="ptBox col-12"
            align="right"
            disabled
          >
          {{i}}
          </q-input>
        </div>
      </div>
      <div class="ptBox col-5" style="padding: 0 10px">
        <div class="row" style="padding: 15px 0">
          <q-input
            v-for="(pt1_c, i) in pt1"
            v-bind:key="i"
            v-model="pt1[i]"
            class="ptBox col-1"
            align="center"
            maxlength=1
            @keyup="syncPT(1)"
          />
        </div>
      </div>
      <div class="ptBox col-5" style="padding: 0 10px">
        <div class="row" style="padding: 15px 0">
          <q-input
            v-for="(pt2_c, j) in pt2"
            v-bind:key="j"
            v-model="pt2[j]"
            class="ptBox col-1"
            align="center"
            maxlength=1
            @keyup="syncPT(2)"
          />
        </div>
      </div>
    </div>

    
    <div class="row">
      <p class="col-12">pt1: {{pt1_text}}</p>
      <p class="col-12">pt2: {{pt2_text}}</p>
    </div>
    <div class="row">
      pt1: {{pt1}}
      pt2: {{pt2}}
      ct: {{ct}}
    </div>
  </div>
  </q-page>
</template>

<script>
export default {
  data () {
    return {
      charlist: 'abcdefghijklmnopqrstuvwxyz._',
      pt1: [],
      pt2: [],
      ct: [],
      ct_text: '00100 11001'
    }
  },
  methods: {
    parseRawCT(ct_text) {
      let ct_str_list = ct_text.split(/[^01]+/)
      let ct = ct_str_list.map(ct_str => {
        return parseInt(ct_str, 2)
      })
      return ct
    },
    encode_es(str_arr) {
      return str_arr.map(char => {
        if (char=='') return 0
        else return this.charlist.indexOf(char)
      })
    },
    decode_es(arr) {
      return arr.map(num => {
        return this.charlist[num]
      })
    },
    encode_single_es(char) {
      return this.charlist.indexOf(char)
    },
    decode_single_es(idx) {
      return this.charlist[idx]
    },
    updateCT() {
      this.ct = this.parseRawCT(this.ct_text)
      let newPt1 = Array(this.ct.length).fill('')
      let newPt2 = Array(this.ct.length).fill('')
      this.pt1 = newPt1.map((item, i) => {
        return this.pt1[i] ? this.pt1[i] : ''
      })
      this.pt2 = newPt2.map((item, i) => {
        return this.pt2[i] ? this.pt2[i] : ''
      })
    },
    syncPT(from) {
      console.log('syncing from', from, 'to', 3-from)
      if (from === 1) {
        this.pt2 = this.pt1.map((char, i) => {
          if (char == '') return ''
          else {
            let newChar = this.decode_single_es(this.encode_single_es(char)^this.ct[i])
            return newChar ? newChar : '!'
          }
        })
      }
      else if (from === 2) {
        this.pt1 = this.pt2.map((char, i) => {
          if (char == '') return ''
          else {
            let newChar = this.decode_single_es(this.encode_single_es(char)^this.ct[i])
            return newChar ? newChar : '!'
          }
        })
      }
    }
  },
  computed: {
    pt1_text() {
      return this.pt1.map(char => char ? char : '?').join('')
    },
    pt2_text() {
      return this.pt2.map(char => char ? char : '?').join('')
    },
    row_nums() {
      let ptlength = this.pt1.length
      let rows = []
      for (let i = 0; i <= Math.ceil(ptlength/12); i++)
      {
        rows.push(i*12)
      }
      return rows
    }
  },
  watch: {
      /*
    pt: {
      handler: function(newPt, oldPt) {
        console.log('pt change!', newPt, oldPt)
        for (let i = 0; i < newPt.pt1.length; i++) {
          if (newPt.pt1[i] !== oldPt.pt1[i]) {
            console.log('change detected:', i, oldPt.pt1[i], newPt.pt1[i])
            if (newPt.pt1[i] === '') newPt.pt2[i] = ''
            else {
              let newIdx = this.encode_single_es(newPt.pt1[i])^ct[i]
              newPt.pt2[i] = this.decode_single_es(newIdx)
            }
          }
          else if (newPt.pt2[i] !== oldPt.pt2[i]) {
            console.log('change detected:', i, oldPt.pt2[i], newPt.pt2[i])
            if (newPt.pt2[i] === '') newPt.pt1[i] = ''
            else {
              let newIdx = this.encode_single_es(newPt.pt2[i])^ct[i]
              newPt.pt1[i] = this.decode_single_es(newIdx)
            }
          }
        }

        this.pt = newPt
      },
      deep: true
    }
    */
  }
}
</script>

<style>
.ptBox {
  width: 20px;
  border: 1px solid black; 
}
</style>
