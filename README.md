# hashcott.github.io

Trang cá nhân của Nguyễn Đức Hạnh tại **<https://hashcott.github.io>**: các dự án mình đang làm, thông báo khi có bản cập nhật, và thỉnh thoảng vài bài viết.

Build bằng Jekyll có sẵn của GitHub Pages, không dùng theme, không JavaScript. Toàn bộ giao diện nằm trong một file CSS.

## Cấu trúc

```
_config.yml         tên, mô tả, thông tin tác giả
_data/projects.yml  danh sách dự án trên trang chủ
_posts/             bài viết và thông báo cập nhật
_layouts/           default.html (khung trang), post.html (bài viết)
assets/style.css    giao diện, sáng và tối
img/                ảnh dùng trong bài
```

## Thêm nội dung

**Dự án:** thêm một mục vào `_data/projects.yml`:

```yaml
- name: ten-du-an
  url: https://github.com/hashcott/ten-du-an
  site: https://hashcott.github.io/ten-du-an/   # không bắt buộc
  desc: Mô tả một câu.
```

**Thông báo cập nhật:** tạo `_posts/YYYY-MM-DD-ten-bai.md`:

```markdown
---
title: Ten du an 1.2.0
type: update
---

Nội dung...
```

**Bài viết:** giống trên nhưng bỏ dòng `type: update`. Có thể thêm `subtitle:` và `tags: [...]`.

Đường dẫn bài viết có dạng `/YYYY-MM-DD-ten-bai/`.

## Chạy ở máy

Cần Ruby và Bundler:

```sh
bundle install
bundle exec jekyll serve
```

Rồi mở <http://localhost:4000>. Push lên `master` là GitHub Pages tự build lại.
