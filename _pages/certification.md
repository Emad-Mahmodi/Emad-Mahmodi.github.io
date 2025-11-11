---
layout: page
permalink: /Certification/
title: Certification
description: Professional certifications and credentials
nav: true
nav_order: 7
---

<div class="row mt-5 justify-content-center">
    <div class="col-sm-4 mt-3 mt-md-0">
        <a href="{{ '/assets/img/ICCKE.PNG' | relative_url }}" 
           data-lightbox="certifications" 
           data-title="ICCKE Conference Certificate">
            {% include figure.liquid loading="eager" path="assets/img/ICCKE.PNG" class="img-fluid rounded z-depth-1 zoom" %}
        </a>
    </div>
    <div class="col-sm-4 mt-3 mt-md-0">
        <a href="{{ '/assets/img/Teaching Data mining Certification.jpg' | relative_url }}" 
           data-lightbox="certifications" 
           data-title="Data Mining Teaching Certificate">
            {% include figure.liquid loading="eager" path="assets/img/Teaching Data mining Certification.jpg" class="img-fluid rounded z-depth-1 zoom" %}
        </a>
    </div>
    <div class="col-sm-4 mt-3 mt-md-0">
        <a href="{{ '/assets/img/rah.jpeg' | relative_url }}" 
           data-lightbox="certifications" 
           data-title="Project Management Certificate">
            {% include figure.liquid loading="eager" path="assets/img/rah.jpeg" class="img-fluid rounded z-depth-1 zoom" %}
        </a>
    </div>
</div>
<div class="row mt-5 justify-content-center">
    <div class="col-sm-4 mt-3 mt-md-0">
        <a href="{{ '/assets/img/MSc thesis Advisor Certification.jpg' | relative_url }}" 
           data-lightbox="certifications" 
           data-title="ICCKE Conference Certificate">
            {% include figure.liquid loading="eager" path="assets/img/MSc thesis Advisor Certification.jpg" class="img-fluid rounded z-depth-1 zoom" %}
        </a>
    </div>

</div>

<!-- Lightbox Script + Custom Clean Style -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/lightbox2/2.11.4/js/lightbox-plus-jquery.min.js"></script>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/lightbox2/2.11.4/css/lightbox.min.css">

<script>
  lightbox.option({
    'resizeDuration': 300,
    'fadeDuration': 300,
    'imageFadeDuration': 300,
    'wrapAround': true,
    'alwaysShowNavOnTouchDevices': true,
    'disableScrolling': false,
    'albumLabel': "Image %1 of %2",

    // کاملاً شفاف + صفحه اصلی دیده شود
    'positionFromTop': 50,
    'maxWidth': '95%',
    'maxHeight': '95%'
  })
</script>

<style>
  /* پس‌زمینه کاملاً شفاف + بلور صفحه اصلی */
  #lightboxOverlay {
    background: rgba(255, 255, 255, 0.85) !important;
    backdrop-filter: blur(8px) !important;
    -webkit-backdrop-filter: blur(8px) !important;
  }

  /* حذف کامل پس‌زمینه مشکی دور عکس */
  #lightbox {
    background: transparent !important;
    box-shadow: none !important;
  }

  /* دکمه بستن کوچک و زیبا در گوشه بالا-راست */
  .lb-close {
    background: rgba(0,0,0,0.6) !important;
    border-radius: 50% !important;
    width: 36px !important;
    height: 36px !important;
    opacity: 0.9 !important;
  }

  .lb-close:before {
    content: "×";
    color: white;
    font-size: 28px;
    font-weight: 300;
    line-height: 36px;
  }

  /* هاور روی عکس = بزرگ‌نمایی ملایم */
  .zoom {
    transition: transform 0.3s ease;
  }
  .zoom:hover {
    transform: scale(1.05);
    z-index: 10;
  }

  /* مخفی کردن متن زیرین و کپشن */
  .lb-dataContainer {
    display: none !important;
  }
  .lb-caption, .lb-number {
    display: none !important;
  }
</style>