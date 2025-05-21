# webappCam
기존의 https://github.com/pocky75/webApp 확장버전

# 1차 확장 기능☝️
웹브라우저에서 파일을 선택해서 서버로 전송하고,
서버는 이미지 파일을 /static/uploads 폴더에 저장함
addbook.txt 파일에 내가 적은 내용저장
![스크린샷 2025-05-21 162344](https://github.com/user-attachments/assets/4d2bf0a2-8696-42df-87d2-42bb60da36e4)

# 추가 코드
1️⃣ app.py에서 생일 입력란과 사진 파일명 저장 추가
```
@app.route('/add', methods=['POST'])
def add_contact():
    name = request.form['pyname']
    phone = request.form['pyphone']
    birthday = request.form['pybirthday']  # 생일 입력 추가

    # 사진 업로드 처리
    photo = request.files['pyphoto']
    photo_filename = photo.filename if photo.filename else 'default.png'  # 기본값 설정
    if photo and photo_filename != 'default.png':
        os.makedirs(app.config['UPLOAD_FOLDER'], exist_ok=True)  # 폴더 생성
        photo.save(os.path.join(app.config['UPLOAD_FOLDER'], photo_filename))

    with open('addbook.txt', 'a', newline='', encoding='utf-8') as file:
        writer = csv.writer(file)
        writer.writerow([name, phone, birthday, photo_filename])  # 사진 파일명 저장 추가

    # 저장 후 입력 페이지로 리다이렉트
    return render_template('index.html', uploaded_photo=url_for('static', filename=f'images/{photo_filename}'))
```
2️⃣ index.html에서는 이름, 전화번호 아래 생일, 사진, 추가 코드를 추가함.
아래 {% if uploaded_photo %} 코드는 uploaded_photo라는 
변수가 존재하고 값이 있을 경우에만 아래의 HTML 코드를 렌더링하고,
{% endif %}는 조건문을 닫는 역할을 하며,
{{ uploaded_photo }} 코드는 uploaded_photo 변수의 값을 HTML에 삽입한 뒤에
uploaded_photo라는 변수가 flask 백엔드에서 전달되었을 때만 업로드된 사진을 표시하는 역할을 한다.
```
    <h1>Address Book</h1>
    <form action="/add" method="POST" enctype="multipart/form-data">
        <label for="pyname">이름:</label>
        <input type="text" id="pyname" name="pyname" required>
        <br><br>
        <label for="pyphone">전화번호:</label>
        <input type="text" id="pyphone" name="pyphone" required>
        <br><br>
        <label for="pybirthday">생일:</label>
        <input type="date" id="pybirthday" name="pybirthday" required>
        <br><br>
        <label for="pyphoto">사진:</label>
        <input type="file" id="pyphoto" name="pyphoto" accept="image/*">
        <br><br>
        <button type="submit">추가</button>
    </form>

    {% if uploaded_photo %}
    <h2>업로드된 사진:</h2>
    <img src="{{ uploaded_photo }}" alt="Uploaded Photo" style="max-width: 300px; max-height: 300px;">
    {% endif %}
```
# 실행 결과
uploads폴더로 저장하게되면 실행 창에서 저장을 했을 때 사진이 안 보임
static/images폴더를 만들어 images파일에 저장되도록 변경
![스크린샷 2025-05-21 162943](https://github.com/user-attachments/assets/856777c9-76dd-4db6-ac1a-825c1fb54e0c)
![스크린샷 2025-05-21 162949](https://github.com/user-attachments/assets/ffa8cc6f-c0ed-4564-a8c5-a82fa50b4563)

이렇게 파일을 만든 뒤 텍스트를 넣은 뒤 저장을 누르면 아래 사진처럼 사진이 보임
![스크린샷 2025-05-21 161913](https://github.com/user-attachments/assets/1c64746c-b837-4d11-ac2b-4b8de20e2ce3)

# 2차 확장 기능✌️
- 1차 확장 기능과 같이 추가로 웹캠으로 촬영하는 기능 추가

# 추가 코드
1️⃣ app.py
```
@app.route('/capture', methods=['POST'])
def capture():
    if 'webcam_photo' in request.files:
        photo = request.files['webcam_photo']
        photo_path = os.path.join(app.config['UPLOAD_FOLDER'], 'webcam_photo.png')
        photo.save(photo_path)
        return jsonify({'message': 'Photo saved successfully!', 'photo_path': photo_path})
    return jsonify({'error': 'No photo uploaded'}), 400
```
2️⃣ index.html
HTML에 웹캠 화면과 촬영 버튼을 추가하고, 
JavaScript를 사용해 웹캠에서 이미지를 캡처한 후 서버로 전송하는 기능을 구현.
서버에서 flask를 사용해 이미지를 저장하도록 처리.
```
    <h2>웹캠 촬영</h2>
    <video id="webcam" autoplay playsinline style="max-width: 300px; max-height: 300px;"></video>
    <canvas id="canvas" style="display: none;"></canvas>
    <br>
    <button id="captureButton">촬영</button>

    <script>
        const video = document.getElementById('webcam');
        const canvas = document.getElementById('canvas');
        const captureButton = document.getElementById('captureButton');

        // 웹캠 활성화
        navigator.mediaDevices.getUserMedia({ video: true })
            .then(stream => {
                video.srcObject = stream;
            })
            .catch(err => {
                console.error('웹캠을 활성화할 수 없습니다:', err);
            });

        // 촬영 버튼 클릭 이벤트
        captureButton.addEventListener('click', () => {
            const context = canvas.getContext('2d');
            canvas.width = video.videoWidth;
            canvas.height = video.videoHeight;
            context.drawImage(video, 0, 0, canvas.width, canvas.height);

            // 캡처한 이미지를 서버로 전송
            canvas.toBlob(blob => {
                const formData = new FormData();
                formData.append('webcam_photo', blob, 'webcam_photo.png');

                fetch('/capture', {
                    method: 'POST',
                    body: formData
                })
                .then(response => response.json())
                .then(data => {
                    alert('사진이 저장되었습니다!');
                    location.reload();
                })
                .catch(err => {
                    console.error('사진 저장 중 오류 발생:', err);
                });
            }, 'image/png');
        });
    </script>
```
# 실행 화면
![image](https://github.com/user-attachments/assets/c30fee6a-3e5f-4184-b5b1-559b68a2cc50)
촬영 버튼 누르면 아래처럼 저장되었다고 메세지가 나오고,
images에 사진이 저장된다.
![스크린샷 2025-05-21 165802](https://github.com/user-attachments/assets/389bea80-27ae-4352-bf5a-47b8ad566175)
![스크린샷 2025-05-21 170110](https://github.com/user-attachments/assets/d0c3af68-29e6-4a6a-a13c-63e7dbbc6bd4)

# 2차 최종 코드
1️⃣ app.py
```
from flask import Flask, render_template, request, redirect, url_for, send_from_directory, jsonify
import csv
import os

app = Flask(__name__)
UPLOAD_FOLDER = 'static/images'
app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER

@app.route('/')
def index():
    return render_template('index.html')


@app.route('/uploads/<filename>')
def uploaded_file(filename):
    return send_from_directory(app.config['UPLOAD_FOLDER'], filename)


@app.route('/add', methods=['POST'])
def add_contact():
    name = request.form['pyname']
    phone = request.form['pyphone']
    birthday = request.form['pybirthday']  # 생일 입력 추가

    # 사진 업로드 처리
    photo = request.files['pyphoto']
    photo_filename = photo.filename if photo.filename else 'default.png'  # 기본값 설정
    if photo and photo_filename != 'default.png':
        os.makedirs(app.config['UPLOAD_FOLDER'], exist_ok=True)  # 폴더 생성
        photo.save(os.path.join(app.config['UPLOAD_FOLDER'], photo_filename))

    with open('addbook.txt', 'a', newline='', encoding='utf-8') as file:
        writer = csv.writer(file)
        writer.writerow([name, phone, birthday, photo_filename])  # 사진 파일명 저장 추가

    # 저장 후 입력 페이지로 리다이렉트
    return render_template('index.html', uploaded_photo=url_for('static', filename=f'images/{photo_filename}'))


@app.route('/capture', methods=['POST'])
def capture():
    if 'webcam_photo' in request.files:
        photo = request.files['webcam_photo']
        photo_path = os.path.join(app.config['UPLOAD_FOLDER'], 'webcam_photo.png')
        photo.save(photo_path)
        return jsonify({'message': 'Photo saved successfully!', 'photo_path': photo_path})
    return jsonify({'error': 'No photo uploaded'}), 400


if __name__ == '__main__':
    app.run(debug=True)
```
2️⃣ index.html
```
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Address Book</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}">
</head>
<body>
    <h1>Address Book</h1>
    <form action="/add" method="POST" enctype="multipart/form-data">
        <label for="pyname">이름:</label>
        <input type="text" id="pyname" name="pyname" required>
        <br><br>
        <label for="pyphone">전화번호:</label>
        <input type="text" id="pyphone" name="pyphone" required>
        <br><br>
        <label for="pybirthday">생일:</label>
        <input type="date" id="pybirthday" name="pybirthday" required>
        <br><br>
        <label for="pyphoto">사진:</label>
        <input type="file" id="pyphoto" name="pyphoto" accept="image/*">
        <br><br>
        <button type="submit">추가</button>
    </form>

    {% if uploaded_photo %}
    <h2>업로드된 사진:</h2>
    <img src="{{ uploaded_photo }}" alt="Uploaded Photo" style="max-width: 300px; max-height: 300px;">
    {% endif %}

    <h2>웹캠 촬영</h2>
    <video id="webcam" autoplay playsinline style="max-width: 300px; max-height: 300px;"></video>
    <canvas id="canvas" style="display: none;"></canvas>
    <br>
    <button id="captureButton">촬영</button>

    <script>
        const video = document.getElementById('webcam');
        const canvas = document.getElementById('canvas');
        const captureButton = document.getElementById('captureButton');

        // 웹캠 활성화
        navigator.mediaDevices.getUserMedia({ video: true })
            .then(stream => {
                video.srcObject = stream;
            })
            .catch(err => {
                console.error('웹캠을 활성화할 수 없습니다:', err);
            });

        // 촬영 버튼 클릭 이벤트
        captureButton.addEventListener('click', () => {
            const context = canvas.getContext('2d');
            canvas.width = video.videoWidth;
            canvas.height = video.videoHeight;
            context.drawImage(video, 0, 0, canvas.width, canvas.height);

            // 캡처한 이미지를 서버로 전송
            canvas.toBlob(blob => {
                const formData = new FormData();
                formData.append('webcam_photo', blob, 'webcam_photo.png');

                fetch('/capture', {
                    method: 'POST',
                    body: formData
                })
                .then(response => response.json())
                .then(data => {
                    alert('사진이 저장되었습니다!');
                    location.reload();
                })
                .catch(err => {
                    console.error('사진 저장 중 오류 발생:', err);
                });
            }, 'image/png');
        });
    </script>
</body>
</html>
```
# style.css에서 색상 변경과 가운데 정렬 추가
3️⃣ style.css 코드
```
body {
    font-family: Arial, sans-serif;
    margin: 20px;
    background-color: #2c3e50; /* 어두운 파란색 배경 */
    color: #ecf0f1; /* 밝은 텍스트 색상 */
}
h1, h2 {
    text-align: center;
    color: #4CAF50;
}
form {
    max-width: 400px;
    margin: 0 auto;
    padding: 20px;
    background: #34495e; /* 폼 배경색을 어두운 회색으로 변경 */
    color: #ecf0f1; /* 폼 텍스트 색상 */
    border: 1px solid #ddd;
    border-radius: 8px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}
form label {
    display: block;
    margin-bottom: 8px;
    font-weight: bold;
}
form input, form button {
    width: 100%;
    padding: 10px;
    margin-bottom: 15px;
    border: 1px solid #ccc;
    border-radius: 4px;
    box-sizing: border-box;
    background-color: #2c3e50; /* 입력 필드와 버튼 배경색 */
    color: #ecf0f1; /* 입력 필드와 버튼 텍스트 색상 */
}
form button {
    background-color: #1abc9c; /* 버튼 기본 배경색 */
    color: white;
    font-size: 16px;
    cursor: pointer;
    transition: background-color 0.3s ease;
}

form button:hover {
    background-color: #16a085; /* 버튼 호버 배경색 */
}
video, canvas {
    display: block;
    margin: 20px auto;
    border: 2px solid #ddd;
    border-radius: 8px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}
#captureButton {
    display: block;
    margin: 10px auto;
    padding: 10px 20px;
    background-color: #2196F3;
    color: white;
    border: none;
    border-radius: 4px;
    font-size: 16px;
    cursor: pointer;
    transition: background-color 0.3s ease;
}
#captureButton:hover {
    background-color: #1976D2;
}
```
# 실행 결과
![스크린샷 2025-05-21 170848](https://github.com/user-attachments/assets/6170a17e-fd2b-4012-b9ac-2fa45968e803)

