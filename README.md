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

  





  
