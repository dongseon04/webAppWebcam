# webAppWebcam
이 프로그램은 기존의 https://github.com/dongseon04/webApp.git 을 확장한 것이다.
## 확장기능
1. 웹브라우저에서 파일을 선택해서 서버로 전송
2. 서버는 이미지 파일을 /static/uploads 폴더에 저장혹
3. addbook.txt 파일에 이미지 파일 이름을 저장하게 한다.
   
![image](https://github.com/user-attachments/assets/b606d9d5-2daf-4c99-97b3-9b7edc78d932)

## 주요 수정 내용

`<label for="birthday">Birthday:</label>
      <input type="date" id="birthday" name="pybirthday" required>
      <br><br>
      <label for="photo">Photo:</label>
      <input type="file" id="photo" name="pyphoto" accept="image/*" onchange="previewImage(event)">
      <br><br>
      <img id="photoPreview" src="#" alt="Image Preview" style="display:none; max-width: 200px; max-height: 200px;">
      <br><br>

`
폼 내부에 생일 받는 곳과 파일 선택하는 곳이 추가됨
`  <script>
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
  `
  





  
