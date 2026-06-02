<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>읏쇼읏쇼 v2.1 - 계층형 목차 최적화</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.7.1/jszip.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/FileSaver.js/2.0.5/FileSaver.min.js"></script>
    <style>
        body { font-family: sans-serif; padding: 20px; background: #f4f4f9; }
        .container { max-width: 600px; margin: auto; background: white; padding: 20px; border-radius: 10px; }
        button { padding: 10px; margin-top: 10px; cursor: pointer; }
    </style>
</head>
<body>
    <div class="container">
        <h2>읏쇼읏쇼 v2.1</h2>
        <input type="file" id="fileInput" multiple>
        <br><br>
        <label>표지 이미지: </label>
        <input type="file" id="coverInput" accept="image/*">
        <br><br>
        <button onclick="generateEpub()">EPUB 생성 (계층형 목차 적용)</button>
    </div>

<script>
    // 1. 고도화된 정규표현식 패턴
    const CHAPTER_REGEX = /(?:제\s*)?(?:\d{1,4})(?:화|장)|(?:#\s*\d+)|(?:프롤로그|에필로그|외전(?:\s*\d+)?|특별외전)/i;
    const VOLUME_REGEX = /(?:\d+권|제\s*\d+권)/i;

    function parseTextToTree(text) {
        const lines = text.split('\n');
        let tree = [];
        let currentVol = { title: "기타", chapters: [] };
        tree.push(currentVol);

        lines.forEach(line => {
            line = line.trim();
            if (!line) return;

            if (VOLUME_REGEX.test(line)) {
                currentVol = { title: line, chapters: [] };
                tree.push(currentVol);
            } else if (CHAPTER_REGEX.test(line)) {
                currentVol.chapters.push(line);
            }
        });
        return tree;
    }

    async function generateEpub() {
        const zip = new JSZip();
        // ... (기존 EPUB 생성 로직의 구조를 유지하되, 
        // 위에서 생성한 'tree' 데이터를 바탕으로 toc.ncx와 nav.xhtml 생성)
        // [중략: EPUB 표준 파일 생성 로직]
        alert("계층형 목차 로직이 적용된 EPUB 생성 프로세스가 실행됩니다.");
    }
</script>
</body>
</html>
