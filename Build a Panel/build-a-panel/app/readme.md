# Vài lời nói đầu
Bài này có lẽ là tác giả bị misconfig vì cơ bản mình tìm được 2 lỗi ở đây cơ mà chỉ cần dùng đến 1 và không cần chain vs nhau .-. và mình viết write up này hoàn toàn phụ thuộc vào tâm trạng nên có thể nó sẽ bị dở dở ương ương nếu có gì khó hiểu hoặc không hiểu mn để lại comment để mình sửa nhé! Love you <3

# Challenge
Như mình đã nói ở phần mở đầu bài này có đến 2 lỗ để đục đầu tiên là lỗi `sql injection` và lỗi thứ 2 là `Prototype Pollution` và tác giả cũng đã chia làm 2 challenge nên được thể mình cũng chia đôi luôn :v thế là ăn gian được 1 tí cho đủ KPI =))) [Some music for you while reading !](https://www.youtube.com/watch?v=L3wKzyIN1yk)


![img1](https://github.com/Cl0wnK1n9/diceCTF/blob/main/Build%20a%20Panel/build-a-panel/app/img/Capture.JPG)

Đây là đoạn duy nhất trong đống code của `server.js` không hề sử dụng prestatement và nó dùng để insert dữ liệu vào trong database vậy nên đây là dạng `insert sqli` lỗi này có thể bị lợi dụng để tạo ra một bản ghi có chứa dữ liệu bị giấu ở trong database.
Để dễ hiểu hơn thì mình sẽ đi từ câu query trước
`INSERT INTO widgets (panelid, widgetname, widgetdata) VALUES ('${queryParams['panelid']}', '${queryParams['widgetname']}', '${queryParams['widgetdata']}')`

với `${queryParams['panelid']}` là get parameter với tên panelid. Về cách inject thì khá đơn giản đó là bạn sẽ thêm `'` để break query hiện tại, sau đó tái tạo lại câu lệnh insert đúng và cuối cùng là comment tất cả nhưng phần còn lại để chắc chắn là không có lỗi gì xảy ra.

Mình quyết định sẽ inject vào biến panelid do mình có thể hiển thị tên của widget là flag luôn. Nếu inject vào `widgetdata` thì flag sẽ không hiển thị trực tiếp lên mà bạn phải gọi thông qua đường dẫn khác để xem được giá trị còn nó là đường dẫn nào và sao nó lại không hiển thị trực tiếp thì sao mọi người không thử đọc code nhỉ =))).

Giá trị mình inject vào sẽ trông như thế này `clownking1%27%2C+%28select+flag+from+flag%29+%2C+123%29--+-` 

Và khi đưa vào trong query nó sẽ trông như sau :

`INSERT INTO widgets (panelid, widgetname, widgetdata) VALUES ('clownking1', (select flag from flag) , 123)-- -', 'blabla', 'blabla')`

sau đó để đọc được widget mới được tạo thì như ở trong đường dẫn `/panel/widgets` nó sẽ như này 

![img2](https://github.com/Cl0wnK1n9/diceCTF/blob/main/Build%20a%20Panel/build-a-panel/app/img/Capture2.JPG)

Server sẽ lấy giá trị `panelId` từ cookie và đọc tất cả từ trong database.

Cuối cùng thì mình sẽ gửi url như này `https://build-a-panel.dicec.tf/admin/debug/add_widget?panelid=clownking1%27%2C+%28select+flag+from+flag%29+%2C+123%29--+-&widgetname=123123&widgetdata=123123` và đổi lại cookie nữa là xong 

![img3](https://github.com/Cl0wnK1n9/diceCTF/blob/main/Build%20a%20Panel/build-a-panel/app/img/Capture3.JPG)
