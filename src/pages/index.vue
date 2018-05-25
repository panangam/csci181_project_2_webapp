<template>
  <q-page padding class="row justify-center" style="padding: 15px;">
  <div style="width: 800px; max-width: 90vw;" class="overflow-hidden">
    <div class="row">
      <div class="col">
        <label>
          CT
          <q-input
            v-model.trim="ct_text"
            type="textarea"
            rows=3
            placeholder="Paste CT from Schaefer's file here WITHOUT ANY GARBAGE IN FRONT OR BACK!"
            :max-height=100
          />
        </label>
        <q-btn label="update ct" color="primary" @click="updateCT()"/>
      </div>
    </div>
    
    <div class="row">
      <div class="col-12">
        <label>
          PT Tools
          <q-input 
            v-model.trim="pt_input" 
            type="text" 
            placeholder="Put just PT here"
          />
        </label>
        <q-btn label="load pt1" color="secondary" @click="setPT(1)"/>
        <q-btn label="load pt2" color="secondary" @click="setPT(2)"/>
        <q-btn label="reset pt's" color="negative" @click="resetPT"/>
      </div>
    </div>

    <div class="row">
      <div class="ptBox col-2">
        <div class="row" style="padding: 15px 0">
          <q-input
            v-for="(num, k) in row_nums"
            v-model="row_nums[k]"
            v-bind:key="k"
            class="ptBox col-12"
            align="right"
            readonly
          />
        </div>
      </div>
      <div class="ptBox col-5" style="padding: 0 10px">
        <div class="row" style="padding: 15px 0">
          <q-input
            v-for="(pt1_c, i) in pt1"
            v-bind:key="i"
            ref="pt1box"
            v-model="pt1[i]"
            class="ptBox col-1"
            align="center"
            maxlength=1
            @keyup="event => syncPT(event, 1, i)"
          />
        </div>
      </div>
      <div class="ptBox col-5" style="padding: 0 10px">
        <div class="row" style="padding: 15px 0">
          <q-input
            v-for="(pt2_c, j) in pt2"
            v-bind:key="j"
            ref="pt2box"
            v-model="pt2[j]"
            class="ptBox col-1"
            align="center"
            maxlength=1
            @keyup="event => syncPT(event, 2, j)"
          />
        </div>
      </div>
    </div>

    
    <div class="row">
      <div class="col">
        <label>
          PT1
          <q-input
            type="textarea"
            v-model="pt1_text"
            :max-height="60"
            readonly
          />
        </label>
        <label>
          PT2
          <q-input
            type="textarea"
            v-model="pt2_text"
            :max-height=60
            readonly
          />
        </label>
      </div>
    </div>

    <div class="row debugbox">
      DEBUG<br/>
      pt1: {{pt1}}
      pt2: {{pt2}}
      ct: {{ct}}
      pt_input: {{pt_input}}
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
      pt_input: '',
      ct: [],
      ct_text: '00100 11001'
    }
  },
  methods: {
    parseRawCT(ct_text) {
      let ct_str_list = ct_text.trim().split(/[^01]+/)
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
    syncPT(event, from, pos) {
      console.log('syncing from', from, 'to', 3-from, 'at pos', pos, '; key', event.key)
      if (from === 1) {
        let char = this.pt1[pos]
        if (char == '') { 
          this.$set(this.pt2, pos, '')
        } else {
          let newChar = this.decode_single_es(this.encode_single_es(char)^this.ct[pos])
          this.$set(this.pt2, pos, newChar ? newChar : '!')
        }

        if (/^[a-z._]$/.test(event.key) && this.$refs.pt1box[pos+1]) {
          this.$refs.pt1box[pos+1].select()
        }
        else if (event.key==='Backspace' && this.$refs.pt1box[pos-1]) {
          this.$refs.pt1box[pos-1].select()
        }
      }
      else if (from === 2) {
        let char = this.pt2[pos]
        if (char == '') {
          this.$set(this.pt1, pos, '')
        } else {
          let newChar = this.decode_single_es(this.encode_single_es(char)^this.ct[pos])
          this.$set(this.pt1, pos, newChar ? newChar : '!')
        }

        if (/^[a-z._]$/.test(event.key)&&this.$refs.pt2box[pos+1]) {
          this.$refs.pt2box[pos+1].select()
        }
        else if (event.key==='Backspace' && this.$refs.pt2box[pos-1]) {
          this.$refs.pt2box[pos-1].select()
        }
      }
    },
    syncWholePT(from) {
      if (from==1) {
        this.pt2 = this.pt1.map((c, i) => {
          if (c == '') { 
            return ''
          } else {
            let newChar = this.decode_single_es(this.encode_single_es(c)^this.ct[i])
            return newChar
          }
        })
      } else {
        this.pt1 = this.pt2.map((c, i) => {
          if (c == '') { 
            return ''
          } else {
            let newChar = this.decode_single_es(this.encode_single_es(c)^this.ct[i])
            return newChar
          }
        })
      }
    },
    resetPT() {
      this.pt1 = this.pt1.map(() => '')
      this.pt2 = this.pt2.map(() => '')
    },
    setPT(target) {
      console.log('set PT', target, 'to', this.pt_input)
      if (target==1) {
        this.pt1 = this.pt1.map((c, i) => {
          if (this.pt_input[i]&&/[a-z._]/.test(this.pt_input[i]))
            return this.pt_input[i]
          else
            return ''
        })
        console.log(this.pt1)
      } else {
        this.pt2 = this.pt2.map((c, i) => {
          if (this.pt_input[i]&&/[a-z._]/.test(this.pt_input[i]))
            return this.pt_input[i]
          else
            return ''
        })
      }
      this.syncWholePT(target)
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
      for (let i = 0; i < Math.ceil(ptlength/12); i++)
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
