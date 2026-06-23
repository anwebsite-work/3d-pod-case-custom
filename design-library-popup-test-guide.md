# Design Library Popup Test Guide

Dự án: 3D POD Case Customizer  
Mục tiêu: test luồng khách chọn ảnh từ thư viện có sẵn thông qua popup, chưa cần database, chưa cần Shopify Metaobject, chưa cần app admin riêng.

---

## 1. Mục tiêu test

Hiện customizer đã có luồng chính:

```text
Upload artwork
→ Apply lên preview 3D / mockup
→ Chỉnh scale / rotate / move / flip
→ Update summary
→ Export order JSON / Add to cart
```

Bản test này chỉ thêm một luồng mới:

```text
Choose From Library
→ Mở popup
→ Lọc ảnh theo theme / style / search
→ Chọn một design
→ Apply design đó vào artwork layer hiện tại
→ Vẫn chỉnh scale / rotate / move / flip như ảnh upload
→ Export order JSON có đủ Design ID + Print File + Transform
```

Nguyên tắc quan trọng:

```text
Không viết lại preview hiện tại.
Không thay flow upload cũ.
Không cần database.
Không cần Shopify Metaobject ở giai đoạn test.
Chỉ dùng JSON local / hardcode array để mô phỏng thư viện ảnh.
```

---

## 2. Kiến trúc test đề xuất

Thêm 2 phần mới vào customizer:

```text
1. Design Library Modal
   - Popup hiển thị danh sách design
   - Search
   - Filter theme
   - Filter style
   - Select design

2. Artwork Source Manager
   - source = upload hoặc library
   - cùng đổ về artwork.url
```

Flow chuẩn:

```text
Upload Image
→ state.artwork.source = 'upload'
→ state.artwork.url = uploaded object URL

Choose From Library
→ open modal
→ select design
→ state.artwork.source = 'library'
→ state.artwork.url = design.preview
→ state.artwork.designId = design.id
→ state.artwork.printFile = design.printFile

Preview renderer
→ luôn đọc state.artwork.url

Order JSON
→ lưu state.artwork + state.transform
```

---

## 3. State mẫu nên dùng

Nếu code hiện tại chưa có state rõ ràng, hãy tạo một object trung tâm như sau:

```js
const customizerState = {
  product: {
    id: 'iphone-16-pro-max',
    title: 'iPhone 16 Pro Max',
    type: 'phone-case'
  },

  color: {
    id: 'crystal-clear',
    title: 'Crystal Clear'
  },

  artwork: {
    source: null, // null | 'upload' | 'library'
    file: null,
    url: null,
    designId: null,
    title: null,
    thumbnail: null,
    preview: null,
    printFile: null,
    licenseType: null
  },

  background: {
    source: 'default',
    url: null
  },

  transform: {
    scale: 1,
    rotate: 0,
    x: 0,
    y: 0,
    flipX: false,
    flipY: false
  }
};
```

Điểm cốt lõi là preview không cần biết ảnh đến từ upload hay library. Preview chỉ cần đọc:

```js
customizerState.artwork.url
```

---

## 4. Data thư viện test bằng JSON local

Tạo file hoặc biến JS tạm thời:

```js
const DESIGN_LIBRARY = [
  {
    id: 'july4-eagle-001',
    title: 'Retro Eagle USA',
    theme: 'Independence Day',
    style: 'Vintage',
    productTypes: ['phone-case', 'airpods-case'],
    tags: ['usa', 'eagle', 'flag', 'family'],
    thumbnail: 'https://cdn.shopify.com/s/files/your-thumbnail-1.png',
    preview: 'https://cdn.shopify.com/s/files/your-preview-1.png',
    printFile: 'https://cdn.shopify.com/s/files/your-print-file-1.png',
    status: 'published',
    commercialAllowed: true,
    licenseType: 'created-in-house'
  },
  {
    id: 'christmas-family-001',
    title: 'Christmas Family Crew',
    theme: 'Christmas',
    style: 'Cute',
    productTypes: ['phone-case'],
    tags: ['family', 'holiday', 'matching'],
    thumbnail: 'https://cdn.shopify.com/s/files/your-thumbnail-2.png',
    preview: 'https://cdn.shopify.com/s/files/your-preview-2.png',
    printFile: 'https://cdn.shopify.com/s/files/your-print-file-2.png',
    status: 'published',
    commercialAllowed: true,
    licenseType: 'licensed-asset'
  },
  {
    id: 'dog-mom-pastel-001',
    title: 'Dog Mom Pastel Heart',
    theme: 'Pet Lover',
    style: 'Cute',
    productTypes: ['phone-case', 'airpods-case'],
    tags: ['dog', 'mom', 'pastel', 'heart'],
    thumbnail: 'https://cdn.shopify.com/s/files/your-thumbnail-3.png',
    preview: 'https://cdn.shopify.com/s/files/your-preview-3.png',
    printFile: 'https://cdn.shopify.com/s/files/your-print-file-3.png',
    status: 'published',
    commercialAllowed: true,
    licenseType: 'ai-generated-reviewed'
  },
  {
    id: 'hidden-test-001',
    title: 'Hidden Test Design',
    theme: 'Test',
    style: 'Minimal',
    productTypes: ['phone-case'],
    tags: ['hidden'],
    thumbnail: 'https://cdn.shopify.com/s/files/hidden-thumb.png',
    preview: 'https://cdn.shopify.com/s/files/hidden-preview.png',
    printFile: 'https://cdn.shopify.com/s/files/hidden-print.png',
    status: 'hidden',
    commercialAllowed: true,
    licenseType: 'test'
  },
  {
    id: 'blocked-license-001',
    title: 'Blocked License Design',
    theme: 'Test',
    style: 'Minimal',
    productTypes: ['phone-case'],
    tags: ['blocked'],
    thumbnail: 'https://cdn.shopify.com/s/files/blocked-thumb.png',
    preview: 'https://cdn.shopify.com/s/files/blocked-preview.png',
    printFile: 'https://cdn.shopify.com/s/files/blocked-print.png',
    status: 'published',
    commercialAllowed: false,
    licenseType: 'not-for-commercial-use'
  }
];
```

Trong test, `hidden-test-001` và `blocked-license-001` không được hiển thị ngoài popup.

---

## 5. HTML cần thêm vào customizer

Đặt nút này cạnh khu vực upload artwork hiện tại:

```html
<div class="artwork-source-actions">
  <button type="button" class="artwork-btn" id="uploadArtworkBtn">
    Upload Image
  </button>

  <button type="button" class="artwork-btn" id="openDesignLibraryBtn">
    Choose From Library
  </button>

  <button type="button" class="artwork-btn artwork-btn--danger" id="removeArtworkBtn">
    Remove
  </button>
</div>

<div class="selected-artwork-card" id="selectedArtworkCard" hidden>
  <div class="selected-artwork-card__thumb-wrap">
    <img id="selectedArtworkThumb" alt="Selected artwork" class="selected-artwork-card__thumb">
  </div>

  <div class="selected-artwork-card__content">
    <div class="selected-artwork-card__label">Selected Artwork</div>
    <div class="selected-artwork-card__title" id="selectedArtworkTitle">Not selected</div>
    <div class="selected-artwork-card__meta" id="selectedArtworkMeta"></div>
  </div>
</div>
```

Thêm modal vào cuối body hoặc cuối section customizer:

```html
<div class="design-library-modal" id="designLibraryModal" aria-hidden="true">
  <div class="design-library-modal__overlay" data-close-design-library></div>

  <div class="design-library-modal__panel" role="dialog" aria-modal="true" aria-labelledby="designLibraryTitle">
    <div class="design-library-modal__header">
      <div>
        <h2 id="designLibraryTitle">Choose a Design</h2>
        <p>Pick a ready-made design and apply it to your case.</p>
      </div>

      <button type="button" class="design-library-modal__close" data-close-design-library>
        ×
      </button>
    </div>

    <div class="design-library-modal__filters">
      <input
        type="search"
        id="designLibrarySearch"
        class="design-library-modal__search"
        placeholder="Search designs..."
      >

      <div class="design-library-modal__chips" id="designThemeChips"></div>
      <div class="design-library-modal__chips" id="designStyleChips"></div>
    </div>

    <div class="design-library-modal__grid" id="designLibraryGrid"></div>

    <div class="design-library-modal__footer">
      <div class="design-library-modal__selected" id="designLibrarySelectedText">
        No design selected
      </div>

      <div class="design-library-modal__footer-actions">
        <button type="button" class="design-library-modal__secondary" data-close-design-library>
          Cancel
        </button>

        <button type="button" class="design-library-modal__primary" id="useSelectedDesignBtn" disabled>
          Use This Design
        </button>
      </div>
    </div>
  </div>
</div>
```

---

## 6. CSS modal test

```css
.artwork-source-actions {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin: 12px 0;
}

.artwork-btn {
  border: 1px solid #d7d7d7;
  background: #fff;
  color: #151515;
  border-radius: 12px;
  padding: 10px 14px;
  font-weight: 700;
  cursor: pointer;
}

.artwork-btn:hover {
  border-color: #111;
}

.artwork-btn--danger {
  color: #b42318;
}

.selected-artwork-card {
  display: flex;
  align-items: center;
  gap: 12px;
  border: 1px solid #ececec;
  background: #fafafa;
  border-radius: 16px;
  padding: 12px;
  margin-top: 12px;
}

.selected-artwork-card__thumb-wrap {
  width: 64px;
  height: 64px;
  border-radius: 12px;
  overflow: hidden;
  background: #fff;
  border: 1px solid #e6e6e6;
  flex: 0 0 auto;
}

.selected-artwork-card__thumb {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

.selected-artwork-card__label {
  font-size: 12px;
  opacity: 0.62;
  margin-bottom: 4px;
}

.selected-artwork-card__title {
  font-weight: 800;
}

.selected-artwork-card__meta {
  font-size: 13px;
  opacity: 0.72;
  margin-top: 3px;
}

.design-library-modal {
  position: fixed;
  inset: 0;
  z-index: 9999;
  display: none;
}

.design-library-modal.is-open {
  display: block;
}

.design-library-modal__overlay {
  position: absolute;
  inset: 0;
  background: rgba(0, 0, 0, 0.52);
}

.design-library-modal__panel {
  position: absolute;
  inset: 32px;
  max-width: 1180px;
  margin: auto;
  background: #fff;
  border-radius: 24px;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  box-shadow: 0 24px 80px rgba(0, 0, 0, 0.24);
}

.design-library-modal__header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 20px;
  padding: 22px;
  border-bottom: 1px solid #ededed;
}

.design-library-modal__header h2 {
  margin: 0;
  font-size: 24px;
  line-height: 1.1;
}

.design-library-modal__header p {
  margin: 6px 0 0;
  opacity: 0.68;
}

.design-library-modal__close {
  width: 40px;
  height: 40px;
  border-radius: 999px;
  border: 1px solid #e4e4e4;
  background: #fff;
  font-size: 28px;
  line-height: 1;
  cursor: pointer;
}

.design-library-modal__filters {
  padding: 16px 22px;
  border-bottom: 1px solid #ededed;
}

.design-library-modal__search {
  width: 100%;
  border: 1px solid #dfdfdf;
  border-radius: 14px;
  padding: 13px 14px;
  font-size: 15px;
  outline: none;
}

.design-library-modal__search:focus {
  border-color: #111;
}

.design-library-modal__chips {
  display: flex;
  gap: 8px;
  overflow-x: auto;
  padding-top: 12px;
}

.design-library-chip {
  border: 1px solid #dedede;
  background: #fff;
  border-radius: 999px;
  padding: 8px 12px;
  white-space: nowrap;
  font-size: 13px;
  font-weight: 700;
  cursor: pointer;
}

.design-library-chip.is-active {
  background: #111;
  border-color: #111;
  color: #fff;
}

.design-library-modal__grid {
  padding: 22px;
  overflow-y: auto;
  display: grid;
  grid-template-columns: repeat(4, minmax(0, 1fr));
  gap: 14px;
  flex: 1;
}

.design-card {
  border: 1px solid #ececec;
  background: #fff;
  border-radius: 18px;
  overflow: hidden;
  cursor: pointer;
  transition: transform 180ms ease, border-color 180ms ease, box-shadow 180ms ease;
}

.design-card:hover {
  transform: translateY(-2px);
  border-color: #111;
  box-shadow: 0 10px 26px rgba(0, 0, 0, 0.08);
}

.design-card.is-selected {
  border-color: #111;
  box-shadow: 0 0 0 2px #111 inset;
}

.design-card__image-wrap {
  aspect-ratio: 1 / 1;
  background: #f7f7f7;
}

.design-card__image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

.design-card__body {
  padding: 11px;
}

.design-card__title {
  font-weight: 800;
  font-size: 14px;
  line-height: 1.25;
}

.design-card__meta {
  font-size: 12px;
  opacity: 0.68;
  margin-top: 4px;
}

.design-library-empty {
  grid-column: 1 / -1;
  text-align: center;
  padding: 60px 20px;
  opacity: 0.65;
}

.design-library-modal__footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 20px;
  padding: 16px 22px;
  border-top: 1px solid #ededed;
  background: #fff;
}

.design-library-modal__selected {
  font-weight: 700;
}

.design-library-modal__footer-actions {
  display: flex;
  gap: 10px;
}

.design-library-modal__secondary,
.design-library-modal__primary {
  border-radius: 12px;
  padding: 12px 16px;
  font-weight: 800;
  cursor: pointer;
}

.design-library-modal__secondary {
  border: 1px solid #dfdfdf;
  background: #fff;
  color: #111;
}

.design-library-modal__primary {
  border: 1px solid #111;
  background: #111;
  color: #fff;
}

.design-library-modal__primary:disabled {
  opacity: 0.45;
  cursor: not-allowed;
}

@media (max-width: 767px) {
  .design-library-modal__panel {
    inset: 0;
    border-radius: 0;
  }

  .design-library-modal__header {
    padding: 16px;
  }

  .design-library-modal__header h2 {
    font-size: 20px;
  }

  .design-library-modal__filters {
    padding: 14px 16px;
  }

  .design-library-modal__grid {
    padding: 16px;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 12px;
  }

  .design-library-modal__footer {
    position: sticky;
    bottom: 0;
    align-items: stretch;
    flex-direction: column;
    padding: 12px 16px;
  }

  .design-library-modal__footer-actions {
    width: 100%;
  }

  .design-library-modal__secondary,
  .design-library-modal__primary {
    flex: 1;
  }
}
```

---

## 7. JavaScript modal + filter + select

Gắn script này sau khi DOM customizer đã render.

```js
(function () {
  const modal = document.getElementById('designLibraryModal');
  const openBtn = document.getElementById('openDesignLibraryBtn');
  const closeEls = document.querySelectorAll('[data-close-design-library]');
  const searchInput = document.getElementById('designLibrarySearch');
  const themeChipsWrap = document.getElementById('designThemeChips');
  const styleChipsWrap = document.getElementById('designStyleChips');
  const grid = document.getElementById('designLibraryGrid');
  const selectedText = document.getElementById('designLibrarySelectedText');
  const useBtn = document.getElementById('useSelectedDesignBtn');

  const selectedArtworkCard = document.getElementById('selectedArtworkCard');
  const selectedArtworkThumb = document.getElementById('selectedArtworkThumb');
  const selectedArtworkTitle = document.getElementById('selectedArtworkTitle');
  const selectedArtworkMeta = document.getElementById('selectedArtworkMeta');

  let activeTheme = '';
  let activeStyle = '';
  let tempSelectedDesign = null;

  function getCurrentProductType() {
    return customizerState?.product?.type || 'phone-case';
  }

  function getEligibleDesigns() {
    const productType = getCurrentProductType();

    return DESIGN_LIBRARY.filter((design) => {
      return (
        design.status === 'published' &&
        design.commercialAllowed === true &&
        Array.isArray(design.productTypes) &&
        design.productTypes.includes(productType)
      );
    });
  }

  function uniqueValues(items, key) {
    return [...new Set(items.map((item) => item[key]).filter(Boolean))];
  }

  function createChip(label, activeValue, onClick) {
    const button = document.createElement('button');
    button.type = 'button';
    button.className = 'design-library-chip';
    button.textContent = label || 'All';

    if ((label || '') === activeValue || (!label && !activeValue)) {
      button.classList.add('is-active');
    }

    button.addEventListener('click', onClick);
    return button;
  }

  function renderFilterChips() {
    const eligible = getEligibleDesigns();
    const themes = uniqueValues(eligible, 'theme');
    const styles = uniqueValues(eligible, 'style');

    themeChipsWrap.innerHTML = '';
    styleChipsWrap.innerHTML = '';

    themeChipsWrap.appendChild(
      createChip('', activeTheme, () => {
        activeTheme = '';
        renderFilterChips();
        renderDesignGrid();
      })
    );

    themes.forEach((theme) => {
      themeChipsWrap.appendChild(
        createChip(theme, activeTheme, () => {
          activeTheme = theme;
          renderFilterChips();
          renderDesignGrid();
        })
      );
    });

    styleChipsWrap.appendChild(
      createChip('', activeStyle, () => {
        activeStyle = '';
        renderFilterChips();
        renderDesignGrid();
      })
    );

    styles.forEach((style) => {
      styleChipsWrap.appendChild(
        createChip(style, activeStyle, () => {
          activeStyle = style;
          renderFilterChips();
          renderDesignGrid();
        })
      );
    });
  }

  function getFilteredDesigns() {
    const search = (searchInput.value || '').trim().toLowerCase();

    return getEligibleDesigns().filter((design) => {
      const haystack = [
        design.title,
        design.theme,
        design.style,
        ...(design.tags || [])
      ]
        .filter(Boolean)
        .join(' ')
        .toLowerCase();

      const matchSearch = !search || haystack.includes(search);
      const matchTheme = !activeTheme || design.theme === activeTheme;
      const matchStyle = !activeStyle || design.style === activeStyle;

      return matchSearch && matchTheme && matchStyle;
    });
  }

  function renderDesignGrid() {
    const designs = getFilteredDesigns();
    grid.innerHTML = '';

    if (!designs.length) {
      grid.innerHTML = '<div class="design-library-empty">No designs found.</div>';
      return;
    }

    designs.forEach((design) => {
      const card = document.createElement('button');
      card.type = 'button';
      card.className = 'design-card';
      card.dataset.designId = design.id;

      if (tempSelectedDesign && tempSelectedDesign.id === design.id) {
        card.classList.add('is-selected');
      }

      card.innerHTML = `
        <div class="design-card__image-wrap">
          <img class="design-card__image" src="${design.thumbnail}" alt="${design.title}" loading="lazy">
        </div>
        <div class="design-card__body">
          <div class="design-card__title">${design.title}</div>
          <div class="design-card__meta">${design.theme} · ${design.style}</div>
        </div>
      `;

      card.addEventListener('click', () => {
        tempSelectedDesign = design;
        selectedText.textContent = `Selected: ${design.title}`;
        useBtn.disabled = false;
        renderDesignGrid();
      });

      grid.appendChild(card);
    });
  }

  function openModal() {
    tempSelectedDesign = null;
    selectedText.textContent = 'No design selected';
    useBtn.disabled = true;
    modal.classList.add('is-open');
    modal.setAttribute('aria-hidden', 'false');
    document.documentElement.style.overflow = 'hidden';
    renderFilterChips();
    renderDesignGrid();
  }

  function closeModal() {
    modal.classList.remove('is-open');
    modal.setAttribute('aria-hidden', 'true');
    document.documentElement.style.overflow = '';
  }

  function resetTransformForNewArtwork() {
    customizerState.transform = {
      scale: 1,
      rotate: 0,
      x: 0,
      y: 0,
      flipX: false,
      flipY: false
    };
  }

  function applyLibraryDesign(design) {
    customizerState.artwork = {
      source: 'library',
      file: null,
      url: design.preview,
      designId: design.id,
      title: design.title,
      thumbnail: design.thumbnail,
      preview: design.preview,
      printFile: design.printFile,
      licenseType: design.licenseType
    };

    resetTransformForNewArtwork();

    // IMPORTANT:
    // Đổi tên function bên dưới theo code hiện tại của bạn.
    // Mục tiêu là dùng cùng function mà upload image đang dùng để apply ảnh lên case.
    if (typeof applyArtworkToCase === 'function') {
      applyArtworkToCase(design.preview);
    }

    if (typeof updateTransformControls === 'function') {
      updateTransformControls(customizerState.transform);
    }

    if (typeof updateSummary === 'function') {
      updateSummary();
    }

    if (typeof updateOrderJson === 'function') {
      updateOrderJson();
    }

    updateSelectedArtworkCard();
    closeModal();
  }

  function updateSelectedArtworkCard() {
    const artwork = customizerState.artwork;

    if (!artwork || !artwork.url) {
      selectedArtworkCard.hidden = true;
      return;
    }

    selectedArtworkCard.hidden = false;
    selectedArtworkThumb.src = artwork.thumbnail || artwork.url;
    selectedArtworkTitle.textContent = artwork.title || 'Uploaded artwork';
    selectedArtworkMeta.textContent = artwork.source === 'library'
      ? `Library · ${artwork.designId}`
      : 'Customer upload';
  }

  openBtn?.addEventListener('click', openModal);

  closeEls.forEach((el) => {
    el.addEventListener('click', closeModal);
  });

  searchInput?.addEventListener('input', renderDesignGrid);

  useBtn?.addEventListener('click', () => {
    if (!tempSelectedDesign) return;
    applyLibraryDesign(tempSelectedDesign);
  });

  document.addEventListener('keydown', (event) => {
    if (event.key === 'Escape' && modal.classList.contains('is-open')) {
      closeModal();
    }
  });

  window.updateSelectedArtworkCard = updateSelectedArtworkCard;
})();
```

---

## 8. Kết nối với upload flow hiện tại

Khi khách upload ảnh, cũng nên update state theo cùng format.

Ví dụ:

```js
function handleArtworkUpload(file) {
  const objectUrl = URL.createObjectURL(file);

  customizerState.artwork = {
    source: 'upload',
    file,
    url: objectUrl,
    designId: null,
    title: file.name,
    thumbnail: objectUrl,
    preview: objectUrl,
    printFile: null,
    licenseType: 'customer-upload'
  };

  customizerState.transform = {
    scale: 1,
    rotate: 0,
    x: 0,
    y: 0,
    flipX: false,
    flipY: false
  };

  applyArtworkToCase(objectUrl);
  updateSelectedArtworkCard?.();
  updateSummary?.();
  updateOrderJson?.();
}
```

Nếu code hiện tại đã có function upload khác, không cần thay hết. Chỉ cần thêm đoạn update `customizerState.artwork` vào trong function upload đó.

---

## 9. Kết nối với remove artwork

```js
function removeArtwork() {
  customizerState.artwork = {
    source: null,
    file: null,
    url: null,
    designId: null,
    title: null,
    thumbnail: null,
    preview: null,
    printFile: null,
    licenseType: null
  };

  customizerState.transform = {
    scale: 1,
    rotate: 0,
    x: 0,
    y: 0,
    flipX: false,
    flipY: false
  };

  if (typeof clearArtworkFromCase === 'function') {
    clearArtworkFromCase();
  }

  updateSelectedArtworkCard?.();
  updateSummary?.();
  updateOrderJson?.();
}

const removeArtworkBtn = document.getElementById('removeArtworkBtn');
removeArtworkBtn?.addEventListener('click', removeArtwork);
```

---

## 10. Order JSON builder mẫu

Nếu hiện tại bạn đã có export JSON, hãy thêm phần artwork source.

```js
function buildOrderPayload() {
  return {
    product: {
      id: customizerState.product.id,
      title: customizerState.product.title,
      type: customizerState.product.type
    },

    color: {
      id: customizerState.color.id,
      title: customizerState.color.title
    },

    artwork: {
      source: customizerState.artwork.source,
      title: customizerState.artwork.title,
      url: customizerState.artwork.url,
      designId: customizerState.artwork.designId,
      preview: customizerState.artwork.preview,
      printFile: customizerState.artwork.printFile,
      licenseType: customizerState.artwork.licenseType
    },

    transform: {
      scale: customizerState.transform.scale,
      rotate: customizerState.transform.rotate,
      x: customizerState.transform.x,
      y: customizerState.transform.y,
      flipX: customizerState.transform.flipX,
      flipY: customizerState.transform.flipY
    },

    generatedAt: new Date().toISOString()
  };
}
```

Khi hiển thị export JSON:

```js
function updateOrderJson() {
  const payload = buildOrderPayload();
  const jsonEl = document.getElementById('orderJsonOutput');

  if (jsonEl) {
    jsonEl.textContent = JSON.stringify(payload, null, 2);
  }
}
```

---

## 11. Shopify line item properties mô phỏng

Khi sau này đưa vào Shopify, product form nên có hidden inputs:

```html
<input type="hidden" name="properties[Artwork Source]" id="propArtworkSource">
<input type="hidden" name="properties[Design ID]" id="propDesignId">
<input type="hidden" name="properties[Design Title]" id="propDesignTitle">
<input type="hidden" name="properties[Design Preview]" id="propDesignPreview">
<input type="hidden" name="properties[_Print File]" id="propPrintFile">
<input type="hidden" name="properties[Transform]" id="propTransform">
<input type="hidden" name="properties[_Order JSON]" id="propOrderJson">
```

Function sync:

```js
function syncShopifyProperties() {
  const payload = buildOrderPayload();
  const artwork = customizerState.artwork;
  const transform = customizerState.transform;

  const setValue = (id, value) => {
    const el = document.getElementById(id);
    if (el) el.value = value || '';
  };

  setValue('propArtworkSource', artwork.source || '');
  setValue('propDesignId', artwork.designId || '');
  setValue('propDesignTitle', artwork.title || '');
  setValue('propDesignPreview', artwork.preview || artwork.url || '');
  setValue('propPrintFile', artwork.printFile || '');
  setValue(
    'propTransform',
    `Scale ${transform.scale}, Rotate ${transform.rotate}, X ${transform.x}, Y ${transform.y}, FlipX ${transform.flipX}, FlipY ${transform.flipY}`
  );
  setValue('propOrderJson', JSON.stringify(payload));
}
```

Nên gọi `syncShopifyProperties()` trước khi add to cart.

---

## 12. Custom Summary nên hiển thị gì

Nếu khách upload:

```text
Artwork Source: Customer Upload
Artwork: customer-photo.png
```

Nếu khách chọn library:

```text
Artwork Source: Library
Design: Retro Eagle USA
Design ID: july4-eagle-001
Theme: Independence Day
```

Không nhất thiết show license cho khách. Nhưng trong order JSON vẫn nên lưu `licenseType` để admin trace lại.

---

## 13. Checklist test thủ công

### Test 1: Mở popup

```text
Bấm Choose From Library
→ Popup mở
→ Không bị scroll body phía sau
→ Có search, chips, grid ảnh
```

### Test 2: Chỉ hiện ảnh hợp lệ

```text
Ảnh status = hidden không được hiện
Ảnh commercialAllowed = false không được hiện
Ảnh không thuộc productTypes hiện tại không được hiện
```

### Test 3: Filter theme

```text
Click Independence Day
→ Chỉ hiện ảnh Independence Day
Click All
→ Hiện lại tất cả ảnh hợp lệ
```

### Test 4: Search

```text
Search "dog"
→ Hiện ảnh có title/theme/style/tags chứa dog
Search không có kết quả
→ Hiện No designs found
```

### Test 5: Chọn design

```text
Click một ảnh
→ Card active
→ Footer hiện Selected: [title]
→ Button Use This Design enabled
```

### Test 6: Apply design

```text
Click Use This Design
→ Popup đóng
→ Artwork apply lên case preview
→ Selected artwork card hiện đúng thumbnail/title
→ Summary update đúng
→ Order JSON có source = library
```

### Test 7: Transform sau khi chọn library

```text
Chọn design
→ Chỉnh scale/rotate/move/flip
→ Preview thay đổi đúng
→ Order JSON lưu đúng transform
```

### Test 8: Đổi từ library sang upload

```text
Chọn design từ library
→ Upload ảnh mới
→ Artwork source chuyển thành upload
→ Design ID bị clear
→ Preview dùng ảnh upload mới
```

### Test 9: Đổi từ upload sang library

```text
Upload ảnh
→ Chọn ảnh từ library
→ Artwork source chuyển thành library
→ File upload cũ không còn là artwork active
→ Preview dùng ảnh library
```

### Test 10: Remove artwork

```text
Click Remove
→ Artwork bị clear
→ Preview không còn artwork
→ Selected artwork card ẩn
→ Order JSON artwork source = null
```

---

## 14. Những lỗi dễ gặp

### Lỗi 1: Chỉ đổi ảnh ngoài UI nhưng không đổi state

Sai vì order JSON vẫn không có design đã chọn.

Phải luôn update:

```js
customizerState.artwork
```

### Lỗi 2: Lưu mỗi tên design

Không đủ. Phải lưu ít nhất:

```text
Design ID
Design Title
Preview URL
Print File URL
Artwork Source
Transform
```

### Lỗi 3: Ảnh hidden vẫn hiện ngoài popup

Filter bắt buộc:

```js
design.status === 'published' && design.commercialAllowed === true
```

### Lỗi 4: Chọn ảnh mới nhưng transform cũ vẫn giữ

Nên reset transform khi đổi artwork để tránh ảnh mới bị lệch hoặc phóng quá lớn.

### Lỗi 5: Load ảnh full-size trong grid

Grid chỉ dùng thumbnail. Preview/print file chỉ dùng khi apply hoặc fulfillment.

---

## 15. Lộ trình sau khi test ổn

### Phase 1: Local JSON

```text
Dùng mảng DESIGN_LIBRARY hardcode
Test popup/filter/select/apply/order JSON
```

### Phase 2: JSON file riêng

```text
Tách DESIGN_LIBRARY ra file design-library.json
Fetch file này khi mở popup
```

### Phase 3: Shopify Metaobject

```text
Admin tạo design trong Shopify
Theme render JSON từ metaobject
Popup dùng data thật
```

### Phase 4: App admin riêng

```text
Bulk upload design
License manager
Approve/reject
Product assignment
Search nâng cao
Analytics design bán chạy
```

---

## 16. Kết luận kỹ thuật

Logic đúng cho test là:

```text
Library popup không phải hệ thống riêng biệt.
Nó chỉ là một cách khác để set artwork.url.
Upload và Library cùng chảy về một artwork layer.
Preview, transform, summary và order JSON dùng chung pipeline.
```

Cấu trúc cốt lõi:

```text
Upload Image
→ artwork.source = upload
→ artwork.url = objectUrl

Choose From Library
→ artwork.source = library
→ artwork.url = design.preview
→ artwork.designId = design.id
→ artwork.printFile = design.printFile

Renderer
→ render artwork.url

Order
→ export artwork + transform
```

Đây là bản test sạch, dễ đưa vào Shopify, dễ nâng cấp sang Metaobject hoặc app riêng sau này.
