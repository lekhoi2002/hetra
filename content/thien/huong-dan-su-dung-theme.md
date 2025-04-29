---
author: Khôi Lê
title: hướng dẫn sử dụng theme
date: 2024-11-29T08:29:23.264Z
description: Mô tả ngắn gọn
image: images/posts/main-image-for-post.webp
thumbnail: images/thumbnails/post-thumb.jpg
images:
    - images/og/post-og-image.png
tags:
    - hugo
    - assets
categories:
    - tutorial
draft: false
preview: /images/mau/giuong-go-oc-cho-gg19-10.webp
---

Nội dung bài viết...

![Ảnh trong nội dung](images/posts/inline-image.png)

Tôi đã xem xét kỹ cấu trúc dự án và nội dung các file bạn cung cấp cho theme Hextra của Hugo. Đây là một theme hiện đại, sử dụng Tailwind CSS và có cấu trúc khá chuẩn mực cho việc phát triển theme Hugo kết hợp với Node.js build tools.

Dưới đây là phân tích chi tiết và một số điểm chúng ta có thể trao đổi để tùy biến theme này:

**Phần 1: Phân Tích Cấu Trúc Dự Án**

Cấu trúc dự án của bạn khá rõ ràng và tuân thủ các quy ước của Hugo và Node.js:

1.  **Root Directory:**
    .hugo_build.lock: File khóa do Hugo tạo ra để quản lý quá trình build.
    .prettierrc: File cấu hình cho Prettier, công cụ định dạng code.
    dev.toml: Cấu hình riêng cho môi trường phát triển Hugo, thường dùng để ghi đè hoặc bổ sung cấu hình trong hugo.yaml khi chạy hugo server. Trong trường hợp này, nó chủ yếu cấu hình cache busting để tự động làm mới CSS/JS khi file nguồn thay đổi.
    hugo.yaml: File cấu hình chính của Hugo cho site (hoặc theme). Định nghĩa các thông số cơ bản, ngôn ngữ, menu, module, và các params tùy chỉnh cho theme Hextra.
    package.json: Định nghĩa các script build (dev, build:css, build) và các gói Node.js cần thiết cho việc phát triển theme (devDependencies như TailwindCSS, PostCSS, Autoprefixer).
    package-lock.json: File tự động sinh ra để khóa phiên bản của các dependencies Node.js, đảm bảo tính nhất quán khi cài đặt lại.
    postcss.config.js: Cấu hình cho PostCSS, định nghĩa các plugin sẽ sử dụng (import, nesting, tailwind, autoprefixer).
    tailwind.config.js: Cấu hình cho Tailwind CSS, nơi bạn định nghĩa prefix (hx-), đường dẫn content, safelist, tùy chỉnh theme (màu sắc, font chữ, breakpoints...), và bật/tắt dark mode.
    taskfile.yaml: File cấu hình cho Task (taskfile.dev), một công cụ chạy tác vụ tương tự Make hoặc npm scripts. Dùng để định nghĩa các lệnh build CSS, chạy dev server, và tải các thư viện bên ngoài (libs).

2.  **assets/ Directory:** Chứa các tài nguyên "thô" cần được xử lý bởi Hugo Pipes hoặc các công cụ Node.js.
    css/:
        styles.css: File CSS chính, import Tailwind và các file CSS thành phần khác. Đây là file đầu vào cho PostCSS.
        tailwind.css: Import các lớp base, components, và utilities của Tailwind.
        custom.css: **Nơi quan trọng để bạn thêm các tùy chỉnh CSS của riêng mình** mà không cần sửa đổi file theme gốc.
        highlight.css & chroma/: Định dạng CSS cho việc tô sáng cú pháp code (syntax highlighting).
        components/: Chứa CSS cho các thành phần UI cụ thể (cards, search, sidebar, navbar,...).
        compiled/main.css: **File CSS đã được biên dịch** bởi PostCSS, chứa tất cả các style cần thiết và sẽ được nhúng vào trang HTML cuối cùng.
    js/: Chứa các file JavaScript cho các chức năng của theme (back-to-top, copy code, menu mobile, search, theme toggle,...).
    json/search-data.json: Template để Hugo tạo file JSON chứa dữ liệu cho chức năng tìm kiếm (FlexSearch).
    lib/: Chứa các thư viện JavaScript/CSS bên thứ ba được tải về (ví dụ: FlexSearch, GLightbox).

3.  **data/ Directory:**
    icons.yaml: Định nghĩa các icon SVG dưới dạng dữ liệu, có thể gọi ra trong template thông qua partial utils/icon.html.

4.  **layouts/ Directory:** Nơi định nghĩa cấu trúc HTML của trang web.
    *.html (root): Các layout cấp cao nhất (ví dụ: hextra-home.html cho trang chủ đặc biệt, index.html cho trang danh sách mặc định).
    _default/: Chứa các template mặc định của Hugo ( baseof.html, list.html, single.html). **baseof.html là template gốc quan trọng nhất.**
        _markup/: Tùy chỉnh cách Hugo render các phần tử Markdown (render hooks), ví dụ: blockquotes (cho alerts), code blocks (cho Mermaid, copy button), headings (thêm anchor link).
    partials/: Chứa các đoạn template có thể tái sử dụng (header, footer, sidebar, TOC, scripts, search,...).
        custom/: **Thư mục quan trọng để bạn override các partial mặc định** (ví dụ footer.html, head-end.html) mà không cần sửa file gốc của theme.
        components/: Partials cho các thành phần UI nhỏ hơn (pager, comments, last updated).
        utils/: Partials chứa các hàm tiện ích dùng trong template (định dạng ngày, lấy title, icon,...).
  

5.  **static/ Directory:** Chứa các file tĩnh sẽ được copy trực tiếp vào thư mục public khi build (ví dụ: site.webmanifest).

**Phần 2: Phân Tích Nội Dung File và Điểm Tùy Biến**

1.  **hugo.yaml:**
    *   **baseURL, title:** Cấu hình cơ bản cho site của bạn.
    *   **languages:** Cấu hình đa ngôn ngữ. Hiện tại chỉ có tiếng Việt (vi).
    *   **module.hugoVersion:** Yêu cầu phiên bản Hugo cụ thể (extended, min 0.134.0).
    *   **markup:** Cấu hình Markdown. unsafe: true cho phép nhúng HTML trực tiếp vào Markdown. enableInlineShortcodes: true cho phép shortcode inline.
    *   **menu.main:** Định nghĩa menu chính hiển thị trên navbar. Bạn có thể thêm, sửa, xóa các mục menu tại đây. Mục type: search là để hiển thị ô tìm kiếm.
    *   **params:** **Khu vực tùy biến quan trọng nhất của Hextra.**
        navbar: Tùy chỉnh logo, hiển thị title, chiều rộng navbar.
        page.width: Chiều rộng mặc định của nội dung trang (normal, wide, full).
        theme: Cấu hình theme sáng/tối (mặc định system, có thể bật/tắt nút chuyển đổi).
        footer: Bật/tắt footer, hiển thị copyright, powered by.
        displayUpdatedDate, dateFormat: Hiển thị ngày cập nhật cuối và định dạng.
        search: Bật/tắt và cấu hình FlexSearch (loại nội dung index, kiểu tokenize). *Lưu ý: Hiện tại enable: false trong file của bạn.*
        blog: Cấu hình hiển thị tags, sắp xếp bài viết trong trang list và hiển thị phân trang trong bài viết đơn lẻ.

2.  **tailwind.config.js:**
    *   **Prefix hx-:** Rất quan trọng khi viết CSS/HTML tùy chỉnh, bạn cần dùng prefix này cho các class của Tailwind (ví dụ hx-text-red-500 thay vì text-red-500).
    *   **content:** Đảm bảo trỏ đúng đến các file chứa class Tailwind, đặc biệt là hugo_stats.json để phát hiện class động từ Hugo.
    *   **theme.colors.primary:** Cách định nghĩa màu chính rất linh hoạt, dựa trên các biến CSS --primary-hue, --primary-saturation, --primary-lightness. Để đổi màu chủ đạo, bạn chỉ cần định nghĩa lại các biến CSS này (ví dụ trong assets/css/custom.css hoặc qua inline style).
    *   **darkMode: ['class', 'html[class~="dark"]']:** Xác nhận chế độ tối được kích hoạt bằng class dark trên thẻ <html>.

3.  **postcss.config.js:** File này định nghĩa pipeline xử lý CSS. Thường bạn không cần sửa trừ khi muốn thêm/bỏ plugin PostCSS.

4.  **package.json & package-lock.json:** Chứa thông tin về các công cụ build (Tailwind, PostCSS...). Bạn sẽ cần chạy npm install để cài đặt chúng nếu muốn build lại CSS hoặc phát triển theme. Các script dev, build:css, build giúp tự động hóa quy trình.

5.  **assets/css/styles.css:** Là file CSS gốc, import các thành phần khác. Nếu muốn thay đổi cấu trúc import CSS, bạn sẽ sửa file này.

6.  **assets/css/custom.css:** **Đây là nơi khuyến nghị để bạn thêm CSS tùy chỉnh.** Các quy tắc CSS ở đây sẽ ghi đè lên các quy tắc mặc định của Hextra hoặc Tailwind nếu có cùng độ ưu tiên hoặc cao hơn. Ví dụ bạn cung cấp cho gallery và GLightbox là rất tốt.

7.  **assets/js/*.js:**
    theme.js: Xử lý logic chuyển đổi theme sáng/tối dựa trên localStorage và tùy chọn hệ thống.
    flexsearch.js: Logic cho chức năng tìm kiếm phía client bằng FlexSearch.
    *   Các file khác cung cấp chức năng tương ứng (copy code, menu, sidebar...). Nếu muốn thay đổi hành vi của các chức năng này, bạn sẽ sửa các file JS tương ứng (yêu cầu kiến thức JavaScript).

8.  **layouts/partials/custom/:** Thư mục này ( footer.html, head-end.html) được thiết kế để bạn đặt các file partial tùy chỉnh. Hugo sẽ ưu tiên các file trong thư mục layouts/partials/custom/ của dự án *của bạn* hơn là của theme. Đây là cách tốt để thêm script tùy chỉnh vào cuối <head> hoặc sửa đổi footer mà không cần fork theme.

9.  **Shortcodes (layouts/shortcodes/)**: Cung cấp các thành phần UI tiện dụng như card, tabs, callout. Bạn có thể tạo shortcode mới hoặc override shortcode của theme bằng cách tạo file cùng tên trong thư mục layouts/shortcodes của dự án.

**Trao Đổi và Hướng Tùy Biến:**

Dựa trên phân tích trên, đây là một số điểm chúng ta có thể thảo luận sâu hơn cho việc tùy biến:

1.  **Màu sắc và Font chữ:**
    *   Bạn muốn thay đổi màu chủ đạo (primary color) không? Chúng ta có thể làm điều này dễ dàng bằng cách định nghĩa lại các biến --primary-hue, --primary-saturation, --primary-lightness trong assets/css/custom.css.
    *   Bạn muốn thay đổi font chữ? Có thể cần cập nhật tailwind.config.js và import font trong CSS.
2.  **Bố cục (Layout):**
    *   Bạn muốn thay đổi chiều rộng mặc định của trang? (Xem params.page.width trong hugo.yaml).
    *   Bạn muốn tùy chỉnh header hoặc footer? Sử dụng layouts/partials/custom/head-end.html và layouts/partials/custom/footer.html hoặc override các partial khác.
3.  **Thành phần UI:**
    *   Bạn muốn thêm/bỏ các mục menu? Chỉnh sửa menu.main trong hugo.yaml.
    *   Bạn muốn tùy chỉnh cách hiển thị blog posts (list, single)? Xem params.blog trong hugo.yaml.
    *   Bạn muốn sửa đổi giao diện của một shortcode (ví dụ: card)? Tạo file override trong layouts/shortcodes của dự án hoặc thêm CSS vào custom.css.
4.  **Chức năng:**
    *   Bạn muốn bật và tinh chỉnh chức năng tìm kiếm? Sửa params.search trong hugo.yaml.
    *   Bạn muốn thêm JavaScript tùy chỉnh? Sử dụng layouts/partials/custom/head-end.html để thêm thẻ <script> hoặc override layouts/partials/scripts.html.
5.  **CSS Nâng cao:**
    *   Bạn cần thêm các lớp tiện ích Tailwind không có sẵn hoặc tùy chỉnh sâu hơn? Sửa tailwind.config.js (cần chạy lại npm run build:css sau khi sửa).
    *   Bạn muốn viết CSS phức tạp không theo dạng utility-first? Viết trong assets/css/custom.css.
6.  **Quy trình làm việc:**
    *   Bạn đã quen với việc sử dụng npm và chạy các script build ( npm run dev, npm run build:css) chưa? Điều này cần thiết khi tùy chỉnh sâu vào CSS/JS.

Hãy cho tôi biết bạn muốn bắt đầu tùy biến phần nào trước, hoặc có yêu cầu cụ thể nào về giao diện/chức năng mà bạn muốn đạt được. Chúng ta có thể đi sâu vào từng file hoặc khu vực cụ thể.