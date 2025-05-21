# webAppWebcam
이 프로그램은 기존의 https://github.com/dongseon04/webApp.git 을 확장한 것이다.
## 확장기능
1. 웹브라우저에서 파일을 선택해서 서버로 전송
2. 서버는 이미지 파일을 /static/uploads 폴더에 저장혹
3. addbook.txt 파일에 이미지 파일 이름을 저장하게 한다.

![image](https://github.com/user-attachments/assets/c073cb1a-e7ae-418f-b1d9-0a4845bb1352)

## 주요 수정 내용

- 폼 내부에 생일 받는 곳과 파일 선택하는 곳이 추가됨
```
<label for="birthday">Birthday:</label>
      <input type="date" id="birthday" name="pybirthday" required>
      <br><br>
      <label for="photo">Photo:</label>
      <input type="file" id="photo" name="pyphoto" accept="image/*" onchange="previewImage(event)">
      <br><br>
      <img id="photoPreview" src="#" alt="Image Preview" style="display:none; max-width: 200px; max-height: 200px;">
      <br><br>
```
- 스크립트가 추가 됨 파일 선택하고 onchange 되었을때 호출 되는 함수
```
 <script>
    function previewImage(event) {
        const photoPreview = document.getElementById('photoPreview');
        const file = event.target.files[0];
        if (file) {
            const reader = new FileReader();
            reader.onload = function(e) {
                photoPreview.src = e.target.result;
                photoPreview.style.display = 'block';
            };
            reader.readAsDataURL(file);
        } else {
            photoPreview.src = '#';
            photoPreview.style.display = 'none';
        }
    }
  </script>
  ```
- 파일내용을 받아서 static/uploads 폴더에 저장하고 파일이름을 addbook.txt에 함께 저장
```
    # 사진 파일 처리
    photo_filename = None
    if photo and allowed_file(photo.filename):
        photo_filename = secure_filename(photo.filename)
        photo.save(os.path.join(app.config['UPLOAD_FOLDER'], photo_filename))

    # CSV 파일에 저장
    with open('addbook.txt', 'a', newline='', encoding='utf-8') as file:
        writer = csv.writer(file)
        writer.writerow([name, phone, birthday, photo_filename])

    return redirect('/')
```
## 실행 화면
![image](https://github.com/user-attachments/assets/b606d9d5-2daf-4c99-97b3-9b7edc78d932)

![image](https://github.com/user-attachments/assets/4d6c5e8e-f107-4e06-8f04-9b2a69281d27)

## 2차 확장 기능
### 웹 카메라로 직접 촬영해서 올리기

1. index.html
```
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Address Book</title>
    <!-- CSS 파일 연결 -->
    <link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}">
</head>
<body>
    <h1>Address Book</h1>
    <form action="/add" method="post" enctype="multipart/form-data">
        <label for="name" class="label-name">Name:</label>
        <input type="text" id="name" name="pyname" required>
        <br><br>
        <label for="phone">Phone:</label>
        <input type="text" id="phone" name="pyphone" required>
        <br><br>
        <label for="birthday">Birthday:</label>
        <input type="date" id="birthday" name="pybirthday" required>
        <br><br>
        <label for="photo">Photo:</label>
        <input type="file" id="photo" name="pyphoto" accept="image/*" onchange="previewPhoto(event)">
        <button type="button" onclick="openCamera()">웹캠 촬영</button>
        <br><br>
        <video id="camera" width="200" autoplay style="display:none;"></video>
        <canvas id="canvas" width="200" style="display:none;"></canvas>
        <img id="photoPreview" src="#" alt="Photo Preview" style="display:none; max-width:200px;"/>
        <br><br>
        <button type="submit">Add Contact</button>        
    </form>
    <script>
        function previewPhoto(event) {
            const input = event.target;
            const preview = document.getElementById('photoPreview');
            if (input.files && input.files[0]) {
                const reader = new FileReader();
                reader.onload = function(e) {
                    preview.src = e.target.result;
                    preview.style.display = 'block';
                }
                reader.readAsDataURL(input.files[0]);
            } else {
                preview.src = '#';
                preview.style.display = 'none';
            }
        }

        function openCamera() {
            const video = document.getElementById('camera');
            const canvas = document.getElementById('canvas');
            video.style.display = 'block';
            navigator.mediaDevices.getUserMedia({ video: true })
                .then(stream => {
                    video.srcObject = stream;
                    video.play();
                    // 촬영 버튼 추가
                    if (!document.getElementById('captureBtn')) {
                        const btn = document.createElement('button');
                        btn.textContent = '촬영';
                        btn.type = 'button';
                        btn.id = 'captureBtn';
                        btn.onclick = function() {
                            capturePhoto(video, canvas);
                        };
                        video.parentNode.insertBefore(btn, video.nextSibling);
                    }
                })
                .catch(err => {
                    alert('웹캠을 사용할 수 없습니다: ' + err);
                });
        }

        function capturePhoto(video, canvas) {
            const context = canvas.getContext('2d');
            canvas.width = video.videoWidth;
            canvas.height = video.videoHeight;
            context.drawImage(video, 0, 0, canvas.width, canvas.height);
            // 미리보기
            const dataURL = canvas.toDataURL('image/png');
            document.getElementById('photoPreview').src = dataURL;
            document.getElementById('photoPreview').style.display = 'block';
            // input[type=file]을 비움
            document.getElementById('photo').value = '';
            // dataURL을 hidden input에 저장
            let hidden = document.getElementById('webcamImage');
            if (!hidden) {
                hidden = document.createElement('input');
                hidden.type = 'hidden';
                hidden.name = 'webcamImage';
                hidden.id = 'webcamImage';
                document.querySelector('form').appendChild(hidden);
            }
            hidden.value = dataURL;
            // 웹캠 끄기
            if (video.srcObject) {
                video.srcObject.getTracks().forEach(track => track.stop());
            }
            video.style.display = 'none';
            canvas.style.display = 'none';
            // 촬영 버튼 제거
            const btn = document.getElementById('captureBtn');
            if (btn) btn.remove();
        }
    </script>
</body>
</html>
```
2. app.py
```
from flask import Flask, render_template, request, redirect
import csv
import os
import base64
from datetime import datetime

app = Flask(__name__)

UPLOAD_FOLDER = 'static/uploads'
os.makedirs(UPLOAD_FOLDER, exist_ok=True)
app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER

# Route for the home page
@app.route('/')
def index():
    return render_template('index.html')

# Route to handle form submission
@app.route('/add', methods=['POST'])
def add_contact():
    name = request.form['pyname']
    phone = request.form['pyphone']
    birthday = request.form['pybirthday']
    photo = request.files.get('pyphoto')
    webcam_image = request.form.get('webcamImage')

    photo_name_to_save = ''

    # 파일 업로드가 있을 때
    if photo and getattr(photo, 'filename', ''):
        photo_filename = os.path.join(app.config['UPLOAD_FOLDER'], photo.filename)
        photo.save(photo_filename)
        photo_name_to_save = photo.filename
    # 웹캠 이미지가 있을 때
    elif webcam_image:
        # 파일명 생성 (예: webcam_20240521_153012.png)
        now_str = datetime.now().strftime('%Y%m%d_%H%M%S')
        photo_name_to_save = f'webcam_{now_str}.png'
        photo_filename = os.path.join(app.config['UPLOAD_FOLDER'], photo_name_to_save)
        # base64 헤더 제거 및 디코딩
        header, data = webcam_image.split(',', 1)
        with open(photo_filename, 'wb') as f:
            f.write(base64.b64decode(data))

    # Save to addbook.txt in CSV format (사진 파일명도 저장)
    with open('addbook.txt', 'a', newline='', encoding='utf-8') as file:
        writer = csv.writer(file)
        writer.writerow([name, phone, birthday, photo_name_to_save])

    return redirect('/')

if __name__ == '__main__':
    app.run(debug=True)
```
## 실행화면
  ![image](https://github.com/user-attachments/assets/079030c4-347c-495a-a6bf-a5d5bbf03ba9)

### 저장된 데이터
![image](https://github.com/user-attachments/assets/fba1a858-ff8d-45cc-9801-2cab04c82f4b)

## index.html에 있는 script를 js 폴더로 옮기기
- JavaScript 코드 분리 : index.html에서 JavaScript 코드를 제거하고, static/js/script.js 파일로 이동
- 외부 JavaScript 파일 연결 : <script src="{{ url_for('static', filename='js/script.js') }}"></script>를 추가하여 외부 JavaScript 파일을 불러온다.

## 최종 코드
1. index.html
```
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Address Book</title>
    <!-- CSS 파일 연결 -->
    <link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}">
</head>
<body>
    <h1>Address Book</h1>
    <form action="/add" method="post" enctype="multipart/form-data">
        <label for="name" class="label-name">Name:</label>
        <input type="text" id="name" name="pyname" required>
        <br><br>
        <label for="phone">Phone:</label>
        <input type="text" id="phone" name="pyphone" required>
        <br><br>
        <label for="birthday">Birthday:</label>
        <input type="date" id="birthday" name="pybirthday" required>
        <br><br>
        <label for="photo">Photo:</label>
        <input type="file" id="photo" name="pyphoto" accept="image/*" onchange="previewPhoto(event)">
        <button type="button" onclick="openCamera()">웹캠 촬영</button>
        <br><br>
        <video id="camera" width="200" autoplay style="display:none;"></video>
        <canvas id="canvas" width="200" style="display:none;"></canvas>
        <img id="photoPreview" src="#" alt="Photo Preview" style="display:none; max-width:200px;"/>
        <br><br>
        <button type="submit">Add Contact</button>        
    </form>
    <!-- JavaScript 파일 연결 -->
    <script src="{{ url_for('static', filename='js/script.js') }}"></script>
</body>
</html>
```

2. app.py
```
from flask import Flask, render_template, request, redirect
import csv
import os
import base64
from datetime import datetime

app = Flask(__name__)

UPLOAD_FOLDER = 'static/uploads'
os.makedirs(UPLOAD_FOLDER, exist_ok=True)
app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER

# Route for the home page
@app.route('/')
def index():
    return render_template('index.html')

# Route to handle form submission
@app.route('/add', methods=['POST'])
def add_contact():
    name = request.form['pyname']
    phone = request.form['pyphone']
    birthday = request.form['pybirthday']
    photo = request.files.get('pyphoto')
    webcam_image = request.form.get('webcamImage')

    photo_name_to_save = ''

    # 파일 업로드가 있을 때
    if photo and getattr(photo, 'filename', ''):
        photo_filename = secure_filename(photo.filename)  # 안전한 파일 이름 생성
        photo_path = os.path.join(app.config['UPLOAD_FOLDER'], photo_filename)
        photo.save(photo_path)
        photo_name_to_save = photo_filename
    # 웹캠 이미지가 있을 때
    elif webcam_image:
        # 파일명 생성 (예: webcam_20240521_153012.png)
        now_str = datetime.now().strftime('%Y%m%d_%H%M%S')
        photo_name_to_save = f'webcam_{now_str}.png'
        photo_path = os.path.join(app.config['UPLOAD_FOLDER'], photo_name_to_save)
        # base64 헤더 제거 및 디코딩
        header, data = webcam_image.split(',', 1)
        with open(photo_path, 'wb') as f:
            f.write(base64.b64decode(data))

    # Save to addbook.txt in CSV format (사진 파일명도 저장)
    with open('addbook.txt', 'a', newline='', encoding='utf-8') as file:
        writer = csv.writer(file)
        writer.writerow([name, phone, birthday, photo_name_to_save])

    return redirect('/')

if __name__ == '__main__':
    app.run(debug=True)
```
3. script.js
```
function previewPhoto(event) {
    const input = event.target;
    const preview = document.getElementById('photoPreview');
    if (input.files && input.files[0]) {
        const reader = new FileReader();
        reader.onload = function(e) {
            preview.src = e.target.result;
            preview.style.display = 'block';
        }
        reader.readAsDataURL(input.files[0]);
    } else {
        preview.src = '#';
        preview.style.display = 'none';
    }
}

function openCamera() {
    const video = document.getElementById('camera');
    const canvas = document.getElementById('canvas');
    video.style.display = 'block';
    navigator.mediaDevices.getUserMedia({ video: true })
        .then(stream => {
            video.srcObject = stream;
            video.play();
            // 촬영 버튼 추가
            if (!document.getElementById('captureBtn')) {
                const btn = document.createElement('button');
                btn.textContent = '촬영';
                btn.type = 'button';
                btn.id = 'captureBtn';
                btn.onclick = function() {
                    capturePhoto(video, canvas);
                };
                video.parentNode.insertBefore(btn, video.nextSibling);
            }
        })
        .catch(err => {
            alert('웹캠을 사용할 수 없습니다: ' + err);
        });
}

function capturePhoto(video, canvas) {
    const context = canvas.getContext('2d');
    canvas.width = video.videoWidth;
    canvas.height = video.videoHeight;
    context.drawImage(video, 0, 0, canvas.width, canvas.height);
    // 미리보기
    const dataURL = canvas.toDataURL('image/png');
    document.getElementById('photoPreview').src = dataURL;
    document.getElementById('photoPreview').style.display = 'block';
    // input[type=file]을 비움
    document.getElementById('photo').value = '';
    // dataURL을 hidden input에 저장
    let hidden = document.getElementById('webcamImage');
    if (!hidden) {
        hidden = document.createElement('input');
        hidden.type = 'hidden';
        hidden.name = 'webcamImage';
        hidden.id = 'webcamImage';
        document.querySelector('form').appendChild(hidden);
    }
    hidden.value = dataURL;
    // 웹캠 끄기
    if (video.srcObject) {
        video.srcObject.getTracks().forEach(track => track.stop());
    }
    video.style.display = 'none';
    canvas.style.display = 'none';
    // 촬영 버튼 제거
    const btn = document.getElementById('captureBtn');
    if (btn) btn.remove();
}
```



  
