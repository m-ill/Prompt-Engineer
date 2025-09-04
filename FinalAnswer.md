아래 구조기술사 답안지 텍스트를 손글씨 스타일의 줄노트 HTML로 변환해주세요.

[변환 규칙]
1. "문제:" 다음 내용은 회색 문제 박스에 표시
2. 로마숫자(Ⅰ, Ⅱ, Ⅲ...)는 h2 태그로 변환하고 ▶ 기호 추가
3. **굵은글씨**는 <span class="underline"> 또는 <span class="highlight">로 변환
4. 번호 리스트(1. 2. 3.)는 <div class="handwritten">①②③</div>로 변환
5. 수식: \(...\) 또는 $...$ 형태는 그대로 유지 (MathJax가 처리)
6. <Table Start>...<Table End>는 HTML table로 변환
7. ```코드블록```은 제거하고 내용만 사용
8. 중요 개념은 highlight나 circle 클래스로 강조
9. 첨부된 이미지가 있다면 문제에 해당하는 이미지로 상단에 위치하고 base64 포맷으로 변환하여 입력한다. 
10. SVG 그림, 다이어그램은 완벽하게 반영한다.

[필수 HTML 구조]
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>구조기술사 답안지</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.9/MathJax.js?config=TeX-MML-AM_CHTML"></script>
    <script>
        MathJax.Hub.Config({
            tex2jax: {
                inlineMath: [['$','$'], ['\\(','\\)']],
                displayMath: [['$$','$$'], ['\\[','\\]']]
            }
        });
    </script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Nanum+Pen+Script&family=Gowun+Dodum&display=swap');
        
        body {
            font-family: 'Gowun Dodum', sans-serif;
            line-height: 2.2;
            background: [[fafaf8]];
            color: [[1a1a2e]];
            padding: 20px;
            max-width: 900px;
            margin: 0 auto;
        }
        
        .question-box {
            background: [[f5f5f5]];
            border: 2px solid #333;
            padding: 20px;
            margin-bottom: 30px;
            border-radius: 5px;
        }
        
        .question-title {
            font-weight: bold;
            color: [[0f3460]];
            font-size: 18px;
            margin-bottom: 10px;
        }
        
        .answer-sheet {
            background: white;
            padding: 40px;
            box-shadow: 0 0 20px rgba(0,0,0,0.1);
            position: relative;
            background-image: 
                repeating-linear-gradient(
                    transparent,
                    transparent 35px,
                    [[e8e8e8]] 35px,
                    [[e8e8e8]] 36px
                );
        }
        
        .answer-sheet::before {
            content: '';
            position: absolute;
            left: 60px;
            top: 0;
            bottom: 0;
            width: 2px;
            background: [[ff6b6b]];
        }
        
        h1 {
            font-family: 'Nanum Pen Script', cursive;
            font-size: 28px;
            color: [[0f3460]];
            text-align: center;
            margin-bottom: 30px;
            text-decoration: underline;
            text-underline-offset: 8px;
        }
        
        h2 {
            font-size: 20px;
            color: [[16213e]];
            margin: 25px 0 15px 0;
            padding-left: 70px;
            position: relative;
        }
        
        h2::before {
            content: '▶';
            position: absolute;
            left: 40px;
            color: [[e94560]];
        }
        
        .section {
            padding-left: 70px;
            margin-bottom: 20px;
        }
        
        .formula-box {
            background: [[f0f0f0]];
            border: 1px solid #333;
            padding: 10px;
            margin: 15px 0;
            text-align: center;
            border-radius: 5px;
            font-weight: bold;
        }
        
        .highlight {
            background: linear-gradient(transparent 60%, [[fff3a3]] 60%);
            padding: 0 3px;
        }
        
        .underline {
            text-decoration: underline;
            text-decoration-color: [[e94560]];
            text-decoration-thickness: 2px;
            text-underline-offset: 3px;
        }
        
        .circle {
            display: inline-block;
            border: 2px solid [[0f3460]];
            border-radius: 50%;
            padding: 2px 8px;
            margin: 0 5px;
        }
        
        .note {
            color: [[e94560]];
            font-weight: bold;
        }
        
        ul {
            list-style: none;
            padding-left: 0;
        }
        
        ul li {
            position: relative;
            padding-left: 25px;
            margin-bottom: 10px;
        }
        
        ul li::before {
            content: '•';
            position: absolute;
            left: 0;
            color: [[e94560]];
            font-weight: bold;
            font-size: 20px;
        }
        
        .subsection {
            margin-left: 30px;
            margin-top: 10px;
        }
        
        .arrow {
            color: [[e94560]];
            font-weight: bold;
        }
        
        table {
            border-collapse: collapse;
            margin: 20px 0;
            width: 100%;
        }
        
        th, td {
            border: 1px solid #333;
            padding: 8px;
            text-align: left;
        }
        
        th {
            background: [[f0f0f0]];
            font-weight: bold;
        }
        
        .handwritten {
            font-family: 'Nanum Pen Script', cursive;
            font-size: 24px;
            color: [[0f3460]];
            transform: rotate(-1deg);
        }
        
        .diagram-container {
            text-align: center;
            margin: 20px 0;
            padding: 15px;
            background: [[fafafa]];
            border: 1px solid [[ddd]];
            border-radius: 5px;
        }
    </style>
</head>
<body>
    <!-- 변환된 내용이 여기에 들어갑니다 -->
</body>
</html>

[입력 텍스트]
{여기에 구조기술사 답안지 텍스트를 붙여넣으세요}
