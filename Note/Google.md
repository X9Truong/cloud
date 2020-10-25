### Download file từ google drive tắt chức năng tải xuống

- Ctrl + U tìm đến link có từ `viewerng`

- Paste link vào đây `https://r12a.github.io/app-conversion/` click convert 

- Open link ở ô Character sau đó tìm link dạng `https://doc-0s-5k-apps-viewer.googleusercontent.com/viewer/secure/pdf/9ntm91lgck0jhplteddaakrhuvvov99d/p0h5ed89hp35sivu7en1haabsbghjprq/1570963575000/drive/15111333470000695821/ACFrOgDW4kZ_W_HgNN2VWPdzVd9kCk6e99xpmma2kwy1HABJM60eLmJorchioNCPV39-Nao3y8yOJmSOlaU-lROb8PLOJ1Wkl3McluIt5qnvGzvAaHH5__UyG9EeJLA=` và open link. Enjoy !

### Download file google báo antivirus

`https://drive.google.com/file/d/1anuzyb7GREvGm6SLeN8ob8p8nAuvrflr/view?usp=sharing`

ID: `1anuzyb7GREvGm6SLeN8ob8p8nAuvrflr`

Paste ID to xxx
`https://drive.google.com/uc?export=download&confirm=no_antivirus&id=XXX`

Note: `https://drive.google.com/uc?export=download&id=`

```
let jspdf = document.createElement("script");

jspdf.onload = function () {

    let pdf = new jsPDF();
    let elements = document.getElementsByTagName("img");
    for (let i in elements) {
        let img = elements[i];
        if (!/^blob:/.test(img.src)) {
            continue;
        }
        let can = document.createElement('canvas');
        let con = can.getContext('2d');
        can.width = img.width;
        can.height = img.height;
        con.drawImage(img, 0, 0);
        let imgData = can.toDataURL("image/jpeg", 1.0);
        pdf.addImage(imgData, 'JPEG', 0, 0);
        pdf.addPage();
    }

    pdf.save(document.title.split('.pdf - ')[0]+".pdf");
};

jspdf.src = 'https://gdrive.vip/wp-content/uploads/2020/jspdf.debug.js';
document.body.appendChild(jspdf);
```
* Note: View toàn bộ nội dung file trước khi chạy F12 > console

`https://urlgd.com/vi/download-pdf`


`http://finelybook.com/`




