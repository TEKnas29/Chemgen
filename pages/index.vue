<script >
export default {
  data() {
    return {
      inp: "",
      opisl:"",
      opEng:""
    }
  },
  methods:{
    findChem(){
      let regex = new RegExp("[A-Z][a-z]?\\d*|\\((?:[^()]*(?:\\(.*\\))?[^()]*)+\\)\\d+", 'gm') // https://stackoverflow.com/questions/23602175/regex-for-parsing-chemical-formulas
      let reg1 = new RegExp("\'\'",'gm') //to remove double quates
      let reg2 = new RegExp("\'(.*?)\'",'gm') //values between quotes
      let reg3 = new RegExp('\\\'(Th|A|It|Be|Wh|De|So|Fu|Ha|(\. [A-z][A-z]))\\\'', 'gm') //exception list 
      let reg4 = new RegExp('tert', 'gm') 
      let reg5 = new RegExp('(SN2|sp3|\'SN2\')', 'gm') 
      let reg6 = new RegExp('\\b\\d+\\b', 'gm') //TODO float numbers
      let splchars = new RegExp('(α|β)','gm')
      // TODO ISL part
      let inp = this.inp

      let chk1 = inp.replace(regex,function (x) {
        console.log(x);
        let nreg = new RegExp("^((Ac|Ag|Al|Am|Ar|As|At|Au|B|Ba|Be|Bh|Bi|Bk|Br|C|Ca|Cd|Ce|Cf|Cl|Cm|Co|Cr|Cs|Cu|Ds|Db|Dy|Er|Es|Eu|F|Fe|Fm|Fr|Ga|Gd|Ge|H|He|Hf|Hg|Ho|Hs|I|In|Ir|K|Kr|La|Li|Lr|Lu|Md|Mg|Mn|Mo|Mt|N|Na|Nb|Nd|Ne|Ni|No|Np|O|Os|P|Pa|Pb|Pd|Pm|Po|Pr|Pt|Pu|Ra|Rb|Re|Rf|Rg|Rh|Rn|Ru|S|Sb|Sc|Se|Sg|Si|Sm|Sn|Sr|Ta|Tb|Tc|Te|Th|Ti|Tl|Tm|U|V|W|Xe|Y|Yb|Zn|Zr)\\d*)+$","gm") 
        if(x.match(nreg)){
          return `\'${x}\'`
        }else{
          return `${x}`
        }
      })
      let chk2 = chk1.replace(reg1,'')
      let chk3 = chk2.replace(reg3,function (s) {
        let nreg = new RegExp("\'","g")
        let  z = s.replace(nreg,'')
        return `${z}`
      })
      // ENG  start
      let chk4 = chk3.replace(reg4,function (t) {
        return `<i>${t}</i>`
      })
      let chk5 = chk4.replace(reg5,function (u) {
        let nreg = new RegExp("\'","g")
        let  z = u.replace(nreg,'')
        return `@${z.toLowerCase()};`
      })
      let chk6 = chk5.replace(reg2, function (y){
        let nreg = new RegExp("\'","g")
        let  z = y.replace(nreg,'')
        return `@chem_${z};`
      })
      let chk6a = chk6.replace(reg6,function (t) {
          return `@userf.disp('${t}');`
      })
      let chk6b = chk6a.replace(splchars,function (s) {
        switch (s) {
          case 'α':
              return `@userf.disp('&alpha;');`
            break;
          case 'β':
              return `@userf.disp('&beta;');`
            break;
        
          default:
            break;
        }
        
      })
      this.opEng = `<text ref=EP_text1>${chk6b}</text>`
      // ENG  end
      //ISL start
        let chk7 = chk3.match(reg2)
        let isl = `<!-- ########CHEMSYMB######### -->\n`;
        let isl2 = `<!-- ########ANSPRO######### -->\n`;
        let isl1_temp = ``;
        let isl2_temp = ``;
        let nreg3 = new RegExp("1\,\,","g")
        let nreg4 = new RegExp("\,\}","g")
        
        if (chk7) {
        chk7.forEach((m)=>{
          let nreg = new RegExp('[0-9]','gm')
          let nreg1 = new RegExp("\'","gm")
          let n = m.replace(nreg, function (p) {
              let q = p.replace(nreg1,'')
              return `<sub>${q}</sub>`
            })
          let i = m.replace(nreg1,'')
          let j = n.replace(nreg1,'')

            isl1_temp += `<var name=chem_${i} value="<math><font face=chemsymb>${j}</font></math>">\n`
            
        })
        chk7.forEach((n)=>{
          let nreg = new RegExp('[0-9]','gm')
          let nreg1 = new RegExp("\'","g")
          let nreg2 = new RegExp("\,\,","g")
          let n1 = n.replace(nreg, function (p) {
              let q = p.replace(nreg1,'')
              return `,${q},`
            })
          let i = n1.replace(nreg1,'')
          let j = i.replace(nreg2,'\,')
    
            isl2_temp += `{${j}}\n`

        }) 

        let k = isl2_temp.replace(regex,function (p) {        
              return `${p},1,`
          })
        let l = k.replace(nreg3,'')
        let m = l.replace(nreg4,'\}')
        let n = m.replace('\,)','\}')
        let o = n.replace('(','\{')
        isl2 += o;
        isl = isl + isl1_temp.split("\n")
                    .filter((item, i, allItems) => {
                      return i === allItems.indexOf(item);
                    })
                    .join("\n");
        // console.log(isl1_temp);
        this.opisl = isl + isl2
      }
      //ISL end

    }
  }
}
</script>

<template>
  
  <div class="main">
    <div class="header">
      <div class="titlename">
       <h1>Chemisty Generator</h1> 
      </div>
    </div>
    <hr class="hr">
    <h2 class="h2">Input</h2>
    <div class="inp">
      <div class="inp1">
        <textarea name="inputArea" id="inputArea" class="inputArea" autofocus placeholder="Input text here..." v-model="inp" type="text" @input="findChem"></textarea>
      </div>
    </div>
    <div class="op">
      <div class="op1">
        <h3>ISL:</h3>
        <i>Note: <b>-/+</b> is not included</i>
        <pre class="opArea"><code>{{ opisl }}</code></pre>
      </div>
      <div class="op2">
        <h3>English:</h3>
        <i>Note:Check before direct use still experimental</i>
        <pre class="opArea"><code>{{ opEng }}</code></pre>
      </div>
    </div>
    
  </div>
</template>

<style scoped>
.main{
  background-color: #264653;
  width: 100%;
  margin: 0px;
  padding: 0px;
}
.inputArea{
  background-color: #4ab1cb;
  resize: none;
  border-radius: 10px;
  font-family: Consolas;
  width: 95vw;
  height: 35vh;
}
.opArea > code{
  font-family: Consolas;
  white-space: pre-wrap; 
  word-wrap:normal;
}
.op1 ,.op2{
  background-color: #219EBC;
  width: 700px;
  height: 500px;
  border-radius: 10px;
  word-wrap:normal;
}
.h2{
  text-align: start;
  margin: auto 40px;
}
.hr{
  background-color: black;
  height: 5px;
}
.header{
  text-align: center ;
  height: 4rem;
}
.titlename{
  font-family: 'Courier New', Courier, monospace;
  text-align: center;
  padding-top: 0.5px;
  width: 350px;
  margin: auto;

}
.inp{
  display: flex;
  flex-direction: row;
  flex-wrap: nowrap;
  justify-content: center;
}
.op{
  display: flex;
  flex-direction: row;
  flex-wrap: nowrap;
  justify-content: space-evenly;
}
</style>
