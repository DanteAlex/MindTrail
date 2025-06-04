---
title: Music
date: 2021-08-11
# type: music
# layout: music
# top_img: /img/top-tag.jpg
keywords:
  - 博客
  - 个人博客
  - Ziang Liu
  - blog
  - Ziang Liu's Blog
  - Alex
---

<style >

  #aplayer-author{
    border-bottom: 1px solid #eee;
    padding-bottom: 16px;
  }
  #aplayer-author>.aplayer-btn {
    display: inline-block;
    cursor: pointer;
    color: #777;
  }
  #aplayer-author>.aplayer-btn:hover {
    color: #333
  }
  #aplayer-author>.aplayer-btn-active {
    color: #0077cc
  }
    #aplayer-author>.aplayer-btn-active:hover{
    color: #0077cc
  }


</style>
<div style="display: flex">
  <div style="width: 100px; min-width: 100px">所有歌手</div>
  <div id="aplayer-author"></div>
</div>
<img id="aplayer-loading" style="heigit: 160px; width: 160px; opacity: 0" src="/img/loading.gif"></img>
<div id="aplayer"></div>

<script>
const getJSON = function(url) {
    const promise = new Promise(function(resolve, reject){
        const handler = function() {
            if (this.readyState !== 4) {
                return;
            }
            if (this.status === 200) {
                resolve(this.response);
            } else {
                reject(new Error(this.statusText));
            }
        };
        const client = new XMLHttpRequest();
        client.open("GET", url);
        client.onreadystatechange = handler;
        client.responseType = "json";
        client.setRequestHeader("Accept", "application/json");
        client.send();

    });
    return promise;
};

  const dom = document.getElementById('aplayer-loading');
  setTimeout(() => {
     if(dom) { dom.style.opacity = 100; }
  }, 300)

getJSON("https://cdn.emooa.com/output.json").then(function(json = {}) {

  // 随机排序
  const shuffle = (arr) => {
    let i = arr.length;
    while(i) {
      let j = Math.floor(Math.random() * i--);
      [arr[j], arr[i]] = [arr[i], arr[j]]
    } return arr;
  }

  let audio = shuffle(json.music).map(music => ({...music, 
    url: `https://cdn.emooa.com/${music.url}`,
    pic: `https://cdn.emooa.com/${music.pic}`,
    lrc: `https://cdn.emooa.com/${music.lrc}`
  }));
  
  console.log(1, audio)
 

  // 移除loading
  const loadingDom = document.getElementById('aplayer-loading');
  loadingDom.style.display="none"

/** 加载aplayer */
  const showAplayer = (audio, authorName)=> {
    audio = audio.filter(item => {
      if (authorName) {
        if (authorName === '全部') return true;
        return item.author === authorName;
      }
      return true
    })

    new APlayer({
      container: document.getElementById('aplayer'),
      mini: false,
      autoplay: false,
      // theme: '#FADFA3',
      loop: 'all',
      order: 'list',
      preload: 'auto',
      volume: 0.7,
      mutex: true,
      listFolded: false,
      listMaxHeight: '400px',
      lrcType: 3,
      audio: audio,
    });
  }

  /** 加载歌手 */
  const authorDom = document.getElementById('aplayer-author');
  let author = audio.map(item => item.author);

  var authorTimes = {};
  for (var i = 0; i < author.length; i++) {
    var key = author[i];
    if (authorTimes[key]) {
      authorTimes[key]++;
    } else {
      authorTimes[key] = 1;
    }
  }


  author = Object.entries(authorTimes).sort((a,b) => b[1]-a[1])
  author = [['全部', audio.length],...author];
  author.forEach((item, index) => {
    const div = document.createElement('div');
    div.classList.add('aplayer-btn');
    if(index === 0)  div.classList.add(`aplayer-btn-active`);
    div.style.marginRight = '16px';
    // 添加创建好的文本节点
    div.appendChild(document.createTextNode(`${item[0]}${item[1]=== 1 ?'': `(${item[1]})`}`));
    authorDom.appendChild(div);
  });
  const authorBtn = document.getElementsByClassName("aplayer-btn");
  const authorBtnActive = document.getElementsByClassName("aplayer-btn-active");

  let authorName;
  for (let i in authorBtn) {
    authorBtn[i].onclick = () => {
      authorName=author[i][0];
      showAplayer(audio, authorName);
      if (authorBtnActive[0]) authorBtnActive[0].classList.remove('aplayer-btn-active')
      authorBtn[i].classList.add('aplayer-btn-active');
    }
  }
  showAplayer(audio);
})

</script>
<!--
{% aplayerlist %}{"autoplay": false,"showlrc": 3,"mode": "list", "listMaxHeight": "400px","music": [
  {
    "theme": "#ebd0c2",
    "title": "云烟成雨",
    "author": "房东的猫",
    "url": "https://api.i-meto.com/meting/api?server=tencent&type=url&id=001yYM0I30CzdP&auth=243686f98a14224f0d462fb75e9a3dfe3f3d2b12",
    "pic": "https://api.i-meto.com/meting/api?server=tencent&type=pic&id=004NFJ230yX0Nz&auth=f68522433cb19f7cf34ce99cb9cf7c2ba76ce5a9",
    "lrc": "https://api.i-meto.com/meting/api?server=tencent&type=lrc&id=001yYM0I30CzdP&auth=0b75a8e1f5dbfaddd65cef905563e0b80a162cb2"
  }
]}{% endaplayerlist %} -->
