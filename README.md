# webappCam
기존의 https://github.com/pocky75/webApp 확장버전

# 1차 확장 기능
웹브라우저에서 파일을 선택해서 서버로 전송하고,
서버는 이미지 파일을 /static/uploads 폴더에 저장함
addbook.txt 파일에 내가 적은 내용 저장
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
2️⃣ index.html에서는 이름, 전화번호 아래 생일, 사진, 추가 코드를 추가함
아래 {% if uploaded_photo %} 코드는 uploaded_photo라는 변수가 존재하고 값이 있을 경우에만 아래의 HTML 코드를 렌더링하고,
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

# 2차 확장 기능
