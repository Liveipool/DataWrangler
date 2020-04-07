<template>
  <div>
    <div id="x-spreadsheet-demo"></div>
    <input type="file" name="file" id="pic"/>
  </div>
</template>

<script>
/* global window, document */
import { h } from '../component/element'
import DataProxy from '../core/data_proxy'
import Sheet from '../component/sheet'
import Bottombar from '../component/bottombar'
import { cssPrefix } from '../config'
import { locale } from '../locale/locale'
import '../index.less'
import * as Excel from 'exceljs'

export default {
  name: 'HelloWorld',
  data () {
    return {
    }
  },
  mounted () {
    // 必须要在mounted里注册，不然targetEl = document.querySelector(selectors)是拿不到#x-spreadsheet-demo的
    class Spreadsheet {
      constructor (selectors, options = {}) {
        let targetEl = selectors
        this.options = options
        this.sheetIndex = 1
        this.datas = []
        if (typeof selectors === 'string') {
          targetEl = document.querySelector(selectors)
        }
        this.bottombar = new Bottombar(() => {
          const d = this.addSheet()
          this.sheet.resetData(d)
        }, (index) => {
          const d = this.datas[index]
          this.sheet.resetData(d)
        }, () => {
          this.deleteSheet()
        }, (index, value) => {
          this.datas[index].name = value
        })
        this.data = this.addSheet()
        const rootEl = h('div', `${cssPrefix}`)
          .on('contextmenu', evt => evt.preventDefault())
        // create canvas element
        targetEl.appendChild(rootEl.el)
        this.sheet = new Sheet(rootEl, this.data)
        rootEl.child(this.bottombar.el)
      }

      addSheet (name, active = true) {
        const n = name || `sheet${this.sheetIndex}`
        const d = new DataProxy(n, this.options)
        d.change = (...args) => {
          this.sheet.trigger('change', ...args)
        }
        this.datas.push(d)
        this.bottombar.addItem(n, active)
        this.sheetIndex += 1
        return d
      }

      deleteSheet () {
        const [oldIndex, nindex] = this.bottombar.deleteItem()
        if (oldIndex >= 0) {
          this.datas.splice(oldIndex, 1)
          if (nindex >= 0) this.sheet.resetData(this.datas[nindex])
        }
      }

      loadData (data) {
        // const d = Array.isArray(data) ? data[0] : data;
        const ds = Array.isArray(data) ? data : [data]
        for (let i = 1; i < ds.length; i += 1) {
          const it = ds[i]
          const nd = this.addSheet(it.name, false)
          nd.setData(it)
        }
        this.bottombar.renameItem(0, ds[0].name)
        this.sheet.loadData(ds[0])
        return this
      }

      getData () {
        return this.datas.map(it => it.getData())
      }

      on (eventName, func) {
        this.sheet.on(eventName, func)
        return this
      }

      validate () {
        const { validations } = this.data
        return validations.errors.size <= 0
      }

      change (cb) {
        this.sheet.on('change', cb)
        return this
      }

      static locale (lang, message) {
        locale(lang, message)
      }
    }

    const spreadsheet = (el, options = {}) => new Spreadsheet(el, options)

    if (window) {
      window.x_spreadsheet = spreadsheet
      window.x_spreadsheet.locale = (lang, message) => locale(lang, message)
    }

    const options = {
      mode: 'edit', // edit | read
      showToolbar: true,
      showGrid: true,
      showContextmenu: true,
      view: {
        height: () => document.documentElement.clientHeight,
        width: () => document.documentElement.clientWidth
      },
      row: {
        len: 100,
        height: 25
      },
      col: {
        len: 26,
        width: 100,
        indexWidth: 60,
        minWidth: 60
      },
      style: {
        bgcolor: '#ffffff',
        align: 'left',
        valign: 'middle',
        textwrap: false,
        strike: false,
        underline: false,
        color: '#0a0a0a',
        font: {
          name: 'Helvetica',
          size: 10,
          bold: false,
          italic: false
        }
      }
    }
    const xs = window.x_spreadsheet('#x-spreadsheet-demo', options)
      .loadData([{
        name: 'sheet-test-1',
        freeze: 'B3',
        styles: [
          {
            bgcolor: '#f4f5f8',
            textwrap: true,
            color: '#900b09',
            border: {
              top: ['thin', '#0366d6'],
              bottom: ['thin', '#0366d6'],
              right: ['thin', '#0366d6'],
              left: ['thin', '#0366d6']
            }
          }
        ],
        merges: [
          'C3:D4'
        ],
        rows: {
          1: {
            cells: {
              0: { text: 'testingtesttestetst' },
              2: { text: 'testing' }
            }
          },
          2: {
            cells: {
              0: { text: 'render', style: 0 },
              1: { text: 'Hello' },
              2: { text: 'haha', merge: [1, 1] }
            }
          },
          8: {
            cells: {
              8: { text: 'border test', style: 0 }
            }
          }
        }
      }, { name: 'sheet-test' }]).change((cdata) => {
        console.log('sheet-change', xs.getData())
      })

    xs.on('cell-selected', (cell, ri, ci) => {
      console.log('cell:', cell, ', ri:', ri, ', ci:', ci)
    }).on('cell-edited', (text, ri, ci) => {
      console.log('text:', text, ', ri: ', ri, ', ci:', ci)
    })
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
</style>
