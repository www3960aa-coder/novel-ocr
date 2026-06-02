<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<title>읏쇼읏쇼 v4 - 모바일 안정형 EPUB</title>

<script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.7.1/jszip.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/FileSaver.js/2.0.5/FileSaver.min.js"></script>

<style>
body { font-family:sans-serif; background:#f4f4f9; padding:20px; }
.container { max-width:600px; margin:auto; background:#fff; padding:20px; border-radius:10px; }
button { padding:10px; margin-top:10px; width:100%; }
textarea { width:100%; height:150px; }
</style>
</head>

<body>
<div class="container">

<h2>읏쇼읏쇼 v4 (모바일 최적화)</h2>

<input type="file" id="fileInput">
<br><br>

<button onclick="makeEpub()">EPUB 생성</button>

<br><br>
<textarea id="log"></textarea>

</div>

<script>

const CHAPTER = /(제\s*\d+화|\d+화|#\d+|프롤로그|에필로그|외전|특별외전)/i;
const VOLUME = /(제\s*\d+권|\d+권)/i;

function log(t){
document.getElementById("log").value += t + "\n";
}

function getTitle(text){
let first = text.split("\n").find(l=>l.trim().length>0);
return first ? first.replace(/[^가-힣a-zA-Z0-9]/g,"").slice(0,20) : "ebook";
}

function parse(text){
let lines = text.split("\n");

let tree = [];
let cur = {title:"기타", chapters:[]};
tree.push(cur);

for(let l of lines){
l = l.trim();
if(!l) continue;

if(VOLUME.test(l)){
cur = {title:l, chapters:[]};
tree.push(cur);
}
else if(CHAPTER.test(l)){
cur.chapters.push(l);
}
}

return tree;
}

function makeNCX(tree){
let o = `<?xml version="1.0" encoding="UTF-8"?>
<ncx xmlns="http://www.daisy.org/z3986/2005/ncx/">
<docTitle><text>EPUB</text></docTitle>
<navMap>`;

let i=1;

for(let v of tree){
o += `<navPoint id="v${i}" playOrder="${i}">
<navLabel><text>${v.title}</text></navLabel>
<content src="content.html"/>`;

i++;

for(let c of v.chapters){
o += `<navPoint id="c${i}" playOrder="${i}">
<navLabel><text>${c}</text></navLabel>
<content src="content.html"/>
</navPoint>`;
i++;
}

o += `</navPoint>`;
}

o += `</navMap></ncx>`;
return o;
}

function makeNav(tree){
let h = `<html><body><nav><ol>`;

for(let v of tree){
h += `<li>${v.title}<ol>`;
for(let c of v.chapters){
h += `<li>${c}</li>`;
}
h += `</ol></li>`;
}

h += `</ol></nav></body></html>`;
return h;
}

async function makeEpub(){

let file = document.getElementById("fileInput").files[0];
if(!file){ alert("TXT 넣어"); return; }

let text = await file.text();
let tree = parse(text);

let title = getTitle(text);

log("제목: " + title);

let zip = new JSZip();

zip.file("mimetype","application/epub+zip");

zip.file("META-INF/container.xml",
`<?xml version="1.0"?>
<container version="1.0" xmlns="urn:oasis:names:tc:opendocument:xmlns:container">
<rootfiles>
<rootfile full-path="content.opf" media-type="application/oebps-package+xml"/>
</rootfiles>
</container>`);

zip.file("content.opf",
`<?xml version="1.0"?>
<package version="2.0">
<manifest>
<item id="ncx" href="toc.ncx" media-type="application/x-dtbncx+xml"/>
<item id="nav" href="nav.xhtml" media-type="application/xhtml+xml"/>
<item id="content" href="content.html" media-type="application/xhtml+xml"/>
</manifest>
<spine toc="ncx">
<itemref idref="content"/>
</spine>
</package>`);

zip.file("toc.ncx", makeNCX(tree));
zip.file("nav.xhtml", makeNav(tree));

let content = "<html><body>";

for(let v of tree){
content += `<h1>${v.title}</h1>`;
for(let c of v.chapters){
content += `<h3>${c}</h3>`;
}
}

content += "</body></html>";

zip.file("content.html", content);

zip.generateAsync({type:"blob"}).then(b=>{
saveAs(b, title + ".epub");
log("완료!");
});

}

</script>

</body>
</html>
