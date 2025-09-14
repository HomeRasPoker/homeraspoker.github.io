# homeraspoker.github.io
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Ranking Home Ras Poker 2025</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
<style>
  body {
    font-family: "Trebuchet MS", Arial, sans-serif;
    background: #f5f5f5;
    display: flex;
    flex-direction: column;
    align-items: center;
    margin-top: 30px;
  }

  .topo {
    width: 750px;
    display: flex;
    justify-content: space-between;
    margin-bottom: 15px;
    gap: 10px;
  }

  .btn-salvar, .btn-add, .btn-exportar {
    background: #ffcc00;
    color: #3e2723;
    border: none;
    padding: 10px 15px;
    border-radius: 5px;
    cursor: pointer;
    font-size: 14px;
    font-weight: bold;
    box-shadow: 2px 2px 5px rgba(0,0,0,0.2);
  }
  .btn-salvar:hover, .btn-add:hover, .btn-exportar:hover { background: #ffb300; }

  .ranking {
    background: linear-gradient(135deg, #3e2723 0%, #5d4037 25%, #d7ccc8 50%, #5d4037 75%, #3e2723 100%);
    padding: 20px;
    border-radius: 10px;
    box-shadow: 0 4px 10px rgba(0,0,0,0.2);
    width: 750px;
    margin-bottom: 30px;
    border: 2px solid #ffcc00;
    color: #fff;
    font-weight: bold;
  }

  .ranking h2 {
    text-align: center;
    margin-bottom: 15px;
    color: #ffe082; /* título claro */
    text-transform: uppercase;
    font-weight: bold;
    font-size: 22px;
  }

  .legenda {
    width: 750px;
    display: flex;
    justify-content: center;
    margin-bottom: 10px;
    gap: 20px;
    font-size: 14px;
    color: #000;
  }
  .legenda div { display: flex; align-items: center; gap: 5px; }
  .legenda span { display: inline-block; width: 20px; height: 20px; border-radius: 3px; border: 1px solid #000; }
  .azul { background: #81d4fa; }
  .vermelho { background: #ff8a80; }

  table { width: 100%; border-collapse: collapse; }
  th, td { padding: 6px; text-align: center; border-bottom: 1px solid #a1887f; border-right: 1px solid #a1887f; color: #000; }
  th { background-color: #ffcc00; color: #000; border-bottom: 2px solid #3e2723; }
  tr.pos1_8 td { background: #81d4fa; color: #000; }
  tr.pos9_17 td { background: #ff8a80; color: #000; }
  tr.empty td { color: #555; }
  tr:hover { background-color: rgba(0,0,0,0.1); }

  td[contenteditable="true"] {
    background: rgba(255,236,179,0.3);
    border: 1px dashed #ffcc00;
  }

  .botoes-container { display: flex; justify-content: center; flex-wrap: wrap; gap: 3px; }

  .btn-pontos { 
    padding: 3px 8px; 
    color: white; 
    border: none; 
    border-radius: 4px; 
    font-size: 12px; 
    cursor: pointer; 
    display: inline-block;
  }
  .btn-presenca { background: #4CAF50; }
  .btn-segundo { background: #2196F3; }
  .btn-terceiro { background: #9C27B0; }
  .btn-quarto { background: #FF9800; }
  .btn-quinto { background: #795548; }
  .btn-campeao { background: #FF5722; }

  .animacao-pontos {
    position: absolute;
    font-weight: bold;
    color: #ff5722;
    animation: subirFade 1s forwards;
    pointer-events: none;
    font-size: 14px;
  }
  @keyframes subirFade {
    0% { opacity: 1; transform: translateY(0); }
    100% { opacity: 0; transform: translateY(-25px); }
  }

  #rankingExport {
    display: none;
    background: linear-gradient(135deg, #3e2723 0%, #5d4037 25%, #d7ccc8 50%, #5d4037 75%, #3e2723 100%);
    padding: 20px;
    border-radius: 15px;
    border: 3px solid #ffcc00;
    box-shadow: 10px 10px 25px rgba(0,0,0,0.5);
    width: 750px;
    color: #fff;
    font-weight: bold;
  }
  #rankingExport h2 { text-align: center; color: #ffe082; font-size: 20px; margin-bottom: 15px; }
  #rankingExport table { width: 100%; border-collapse: collapse; }
  #rankingExport th, #rankingExport td { padding: 6px; text-align: center; border: 1px solid #a1887f; color: #000; }
  #rankingExport th { background-color: #ffcc00; color: #000; }
  #rankingExport tr.pos1_8 td { background: #81d4fa; color: #000; }
  #rankingExport tr.pos9_17 td { background: #ff8a80; color: #000; }

</style>
</head>
<body>

<div class="topo">
  <button class="btn-salvar" onclick="salvarRanking()">💾 Salvar Ranking</button>
  <button class="btn-add" onclick="adicionarJogador()">➕ Adicionar Jogador</button>
  <button class="btn-exportar" onclick="exportarTabela()">📷 Exportar Tabela</button>
</div>

<div class="legenda">
  <div><span class="azul"></span> Classificados para a mesa final semestral</div>
  <div><span class="vermelho"></span> Repescagem final</div>
</div>

<div class="ranking" id="rankingContainer">
  <h2>🏆 Ranking Home Ras Poker 2025</h2>
  <table id="rankingTable">
    <tr>
      <th>Posição</th>
      <th>Nome</th>
      <th>Pontos</th>
      <th>Presenças</th>
      <th>Vitórias</th>
      <th>Adicionar Pontos</th>
    </tr>
  </table>
</div>

<div class="ranking" id="rankingExport">
  <h2 id="exportTitulo">🏆 Ranking Home Ras Poker 2025</h2>
  <table id="exportTable">
    <tr>
      <th>Posição</th>
      <th>Nome</th>
      <th>Pontos</th>
      <th>Presenças</th>
      <th>Vitórias</th>
    </tr>
  </table>
</div>

<script>
let nomes = [
  "Vitor Augusto","RacBatis","Briga","Jhow Jhow","Erick","Roque","Nono","Veinho","Rossi","Mario",
  "Piscine","Limoli","Gabriel Vieira","Brunno","Victor Costa","Brian","Caio","Pit","Ale","Zaca",
  "Gui Farias","Leo Conde","Guido","Caio Japa","Chicão","Yoshida","Arrais","Lucas Chaves","Mnafred","Rangel",
  "Samuel","Allan","Dilam","Espeto","Gustavo Alemão","Mateus Grand","Ricardo Cipolli","Pedro Jesus","Boss","Phillipe",
  "William","Mata","Mariana","Ronaldão","David","Dede","Bezerra","Edson Arrais","Henrique","Pedro Bolívia",
  "Rafa Estefam","Danton","Koiti","Fe Gregio","Jorge"
];

let jogadores = [];
if(localStorage.getItem("rankingJogadores")){
  jogadores = JSON.parse(localStorage.getItem("rankingJogadores"));
} else {
  jogadores = nomes.map(nome => ({ nome, pontos: 0, presencas: 0, vitorias: 0 }));
}

function atualizarTabela() {
  const tabela = document.getElementById("rankingTable");
  tabela.innerHTML = `<tr><th>Posição</th><th>Nome</th><th>Pontos</th><th>Presenças</th><th>Vitórias</th><th>Adicionar Pontos</th></tr>`;

  jogadores.sort((a,b) => {
    if(b.vitorias !== a.vitorias) return b.vitorias - a.vitorias;
    if(b.presencas !== a.presencas) return b.presencas - a.presencas;
    return b.pontos - a.pontos;
  });

  jogadores.forEach((j,i)=>{
    const linha = tabela.insertRow();
    if(i<8) linha.classList.add("pos1_8");
    else if(i<17) linha.classList.add("pos9_17");

    linha.insertCell().textContent = i+1;
    let tdNome = linha.insertCell(); tdNome.textContent = j.nome; tdNome.contentEditable="true"; tdNome.onblur=()=>j.nome=tdNome.textContent;
    let tdP = linha.insertCell(); tdP.textContent=j.pontos; tdP.contentEditable="true"; tdP.onblur=()=>j.pontos=parseInt(tdP.textContent)||0;
    let tdPres=linha.insertCell(); tdPres.textContent=j.presencas; tdPres.contentEditable="true"; tdPres.onblur=()=>j.presencas=parseInt(tdPres.textContent)||0;
    let tdVit=linha.insertCell(); tdVit.textContent=j.vitorias; tdVit.contentEditable="true"; tdVit.onblur=()=>j.vitorias=parseInt(tdVit.textContent)||0;

    const celulaBotoes = linha.insertCell();
    const containerBtns = document.createElement("div");
    containerBtns.className = "botoes-container";
    [
      { valor:10, nome:'Presença', classe:'btn-presenca', adicionaPresenca:true },
      { valor:80, nome:'Segundo', classe:'btn-segundo' },
      { valor:60, nome:'Terceiro', classe:'btn-terceiro' },
      { valor:40, nome:'Quarto', classe:'btn-quarto' },
      { valor:20, nome:'Quinto', classe:'btn-quinto' },
      { valor:100, nome:'Campeão', classe:'btn-campeao', adicionaVitoria:true }
    ].forEach(item=>{
      const btn=document.createElement("button");
      btn.textContent=item.nome;
      btn.className=`btn-pontos ${item.classe}`;
      btn.onclick=()=>{
        j.pontos+=item.valor;
        if(item.adicionaPresenca) j.presencas+=1;
        if(item.adicionaVitoria) j.vitorias+=1;
        mostrarAnimacao(tdP,item.valor);
        atualizarTabela();
      };
      containerBtns.appendChild(btn);
    });
    celulaBotoes.appendChild(containerBtns);
  });
}

function mostrarAnimacao(celula, valor){
  const anim = document.createElement("div");
  anim.className = "animacao-pontos";
  anim.textContent = `+${valor}`;
  celula.style.position = "relative";
  celula.appendChild(anim);
  setTimeout(()=> anim.remove(), 1000);
}

function atualizarTabelaExport() {
  const tabela = document.getElementById("exportTable");
  tabela.innerHTML=`<tr><th>Posição</th><th>Nome</th><th>Pontos</th><th>Presenças</th><th>Vitórias</th></tr>`;
  const hoje = new Date();
  document.getElementById("exportTitulo").textContent=`🏆 Ranking Home Ras Poker 2025 - ${hoje.toLocaleDateString("pt-BR")}`;
  jogadores.sort((a,b)=>{
    if(b.vitorias!==a.vitorias) return b.vitorias-a.vitorias;
    if(b.presencas!==a.presencas) return b.presencas-a.presencas;
    return b.pontos-a.pontos;
  });
  jogadores.forEach((j,i)=>{
    const linha=tabela.insertRow();
    linha.insertCell().textContent=i+1;
    linha.insertCell().textContent=j.nome;
    linha.insertCell().textContent=j.pontos;
    linha.insertCell().textContent=j.presencas;
    linha.insertCell().textContent=j.vitorias;
    if(i<8) linha.classList.add("pos1_8"); else if(i<17) linha.classList.add("pos9_17");
  });
}

function salvarRanking(){ localStorage.setItem("rankingJogadores",JSON.stringify(jogadores)); alert("✅ Ranking salvo!"); }
function adicionarJogador(){ let nome=prompt("Digite o nome do novo jogador:"); if(nome&&nome.trim()!==""){ jogadores.push({nome:nome.trim(),pontos:0,presencas:0,vitorias:0}); atualizarTabela(); } }

function exportarTabela(){
  atualizarTabelaExport();
  const container=document.getElementById("rankingExport");
  container.style.display="block";
  html2canvas(container).then(canvas=>{
    const link=document.createElement("a");
    link.download="ranking.png";
    link.href=canvas.toDataURL();
    link.click();
    container.style.display="none";
  });
}

atualizarTabela();
</script>
</body>
</html>
