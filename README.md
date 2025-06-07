<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Image to Video App</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      padding: 20px;
    }
    input[type="file"] {
      margin-bottom: 20px;
    }
    #videoArea img {
      max-width: 80%;
      height: auto;
      display: none;
    }
  </style>
</head>
<body>

  <h1>Image to Video (Preview)</h1>
  <input type="file" id="imageUpload" multiple accept="image/*">
  <br>
  <button onclick="startSlideshow()">Start Preview</button>

  <div id="videoArea"></div>

  <script>
    let images = [];
    let currentIndex = 0;

    document.getElementById('imageUpload').addEventListener('change', function(e) {
      images = Array.from(e.target.files);
      const videoArea = document.getElementById('videoArea');
      videoArea.innerHTML = '';
      images.forEach((imgFile, index) => {
        const img = document.createElement('img');
        img.src = URL.createObjectURL(imgFile);
        img.id = 'img' + index;
        videoArea.appendChild(img);
      });
    });

    function startSlideshow() {
      if (images.length === 0) {
        alert('Please upload some images first!');
        return;
      }

      currentIndex = 0;
      showImage();
    }

    function showImage() {
      const total = images.length;
      for (let i = 0; i < total; i++) {
        document.getElementById('img' + i).style.display = 'none';
      }

      document.getElementById('img' + currentIndex).style.display = 'block';

      currentIndex++;
      if (currentIndex < total) {
        setTimeout(showImage, 1000); // 1 second delay between images
      }
    }
  </script>

</body>
</html>
