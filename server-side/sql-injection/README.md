# SQL injection

## Lab 1: [SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data)

> This lab contains a SQL injection vulnerability in the product category filter. When the user selects a category, the application carries out a SQL query like the following:
>
> ```sql
> SELECT * FROM products WHERE category = 'Gifts' AND released = 1
> ```
>
> To solve the lab, perform a SQL injection attack that causes the application to display one or more unreleased products.

Truy cập vào lab, chúng ta thấy trang web cho phép xem tất cả các sản phẩm hoặc xem sản phẩm theo từng danh mục:

![image](images/lab-1/lab-1.png)

Bài lab yêu cầu chúng ta phải khiến cho ứng dụng hiển thị một hoặc nhiều sản phẩm chưa release.

Dựa vào câu truy vấn mà bài lab cung cấp, có thể hiểu rằng sản phẩm nào đã release thì released = 1, chúng ta cần xem cả những sản phầm chưa release nên cần bypass phần kiểm tra này.

Do vậy, nếu chúng ta thay đổi giá trị của tham số `category` thành `' or 1=1--` sẽ bỏ được phần `AND released = 1` khỏi câu truy vấn. Lúc này câu truy vấn mà ứng dụng thực thi sẽ trở thành:

```sql
SELECT * FROM products WHERE category = '' or 1=1--' AND released = 1
```

Bởi vì `1=1` luôn đúng nên câu truy vấn sẽ trả về tất cả các sản phẩm bao gồm cả những sản phẩm chưa release:

![image](images/lab-1/lab-1-1.png)

## Lab 2: [SQL injection vulnerability allowing login bypass](https://portswigger.net/web-security/sql-injection/lab-login-bypass)

> This lab contains a SQL injection vulnerability in the login function.
>
> To solve the lab, perform a SQL injection attack that logs in to the application as the `administrator` user.

Bên dưới là giao diện tại trang đăng nhập:

![image](images/lab-2/lab-2.png)

Bài lab yêu cầu chúng ta đăng nhập với user là `administrator` nhưng chúng ta không biết password.

Chúng ta có thể dự đoán server sẽ thực hiện câu truy vấn:

```sql
SELECT * FROM users WHERE username = '<username>' AND password = '<password>'
```

Vậy chúng ta sẽ nhập username là `administrator'--` và password bất kì để câu truy vấn trở thành:

```sql
SELECT * FROM users WHERE username = 'administrator'--' AND password = '<password>'
```

Từ đó, chúng ta đăng nhập thành công:

![image](images/lab-2/lab-2-1.png)

## Lab 3: [SQL injection UNION attack, determining the number of columns returned by the query](https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns)

> This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables. The first step of such an attack is to determine the number of columns that are being returned by the query. You will then use this technique in subsequent labs to construct the full attack.
>
> To solve the lab, determine the number of columns returned by the query by performing a SQL injection UNION attack that returns an additional row containing null values.

Truy cập vào lab, chúng ta xem sản phẩm theo danh mục Pets:

![image](images/lab-3/lab-3.png)

Bài lab yêu cầu chúng ta khiến cho câu truy vấn trả về thêm 1 hàng chứa các giá trị null nên có thể thử với payload `' UNION SELECT NULL--` ở tham số `category`.

Chúng ta nhận được lỗi:

![image](images/lab-3/lab-3-1.png)

Từ đó, chúng ta hiểu rằng câu truy vấn gốc lấy ra nhiều hơn 1 cột nên mới gây ra lỗi.

Vậy chúng ta tiếp tục thêm lần lượt giá trị null vào payload để kiểm tra và tới payload `' UNION SELECT NULL, NULL, NULL--` sẽ thành công:

![image](images/lab-3/lab-3-2.png)

## Lab 4: [SQL injection UNION attack, finding a column containing text](https://portswigger.net/web-security/sql-injection/union-attacks/lab-find-column-containing-text)

> This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables. To construct such an attack, you first need to determine the number of columns returned by the query. You can do this using a technique you learned in a previous lab. The next step is to identify a column that is compatible with string data.
>
> The lab will provide a random value that you need to make appear within the query results. To solve the lab, perform a SQL injection UNION attack that returns an additional row containing the value provided. This technique helps you determine which columns are compatible with string data.

Truy cập vào lab, chúng ta sẽ làm tương tự như lab 3 để tìm số cột được trả về từ câu truy vấn:

![image](images/lab-4/lab-4.png)

Xác định được có 3 cột dữ liệu, tiếp đến cần tìm cột có loại dữ liệu phù hợp để chứa được chuỗi `'hJ2Xhz'` nên chúng ta thay lần lượt chuỗi đó vào từng `NULL` trong payload.

Thử thay `'hJ2Xhz'` vào `NULL` đầu tiên nhận được lỗi:

![image](images/lab-4/lab-4-1.png)

Vậy là cột 1 không trùng khớp loại dữ liệu. Giờ chúng ta thay chuỗi `'hJ2Xhz'` vào `NULL` thứ hai sẽ thành công:

![image](images/lab-4/lab-4-2.png)

## Lab 5: [SQL injection attack, querying the database type and version on Oracle](https://portswigger.net/web-security/sql-injection/examining-the-database/lab-querying-database-version-oracle)

> This lab contains a SQL injection vulnerability in the product category filter. You can use a UNION attack to retrieve the results from an injected query.
>
> To solve the lab, display the database version string.
>
> **Hint**
>
> On Oracle databases, every `SELECT` statement must specify a table to select `FROM`. If your `UNION SELECT` attack does not query from a table, you will still need to include the `FROM` keyword followed by a valid table name.
>
> There is a built-in table on Oracle called `dual` which you can use for this purpose. For example: `UNION SELECT 'abc' FROM dual`

Truy cập vào lab, chúng ta có thể đọc các thông tin theo danh mục, ví dụ như Pets:

![image](images/lab-5/lab-5.png)

Bài lab yêu cầu chúng ta phải khiến cho ứng dụng trả về phiên bản của database.

Trước tiên, cần xem xét câu truy vấn ban đầu trả về bao nhiêu cột. Dựa vào hint, mỗi câu lệnh `SELECT` ở Oracle database phải chỉ định một bảng hợp lệ nên chúng ta sẽ tận dụng bảng `dual` và sử dụng payload `' UNION SELECT NULL FROM dual--` để kiểm tra.

![image](images/lab-5/lab-5-1.png)

Đã có lỗi xảy ra, như vậy là câu truy vấn gốc trả về nhiều hơn 1 cột. Chúng ta tiếp tục thêm giá trị null vào để kiểm tra và xác định được số cột là 2.

![image](images/lab-5/lab-5-2.png)

Tiếp theo, cần xem cột nào có thể chứa chuỗi, chúng ta dùng payload `' UNION SELECT 'hehe', NULL FROM dual--` thấy được cột 1 có thể nhận chuỗi.

![image](images/lab-5/lab-5-3.png)

Giờ có thể lấy database version bằng cách sử dụng payload `' UNION SELECT banner, NULL FROM v$version--`.

![image](images/lab-5/lab-5-4.png)
