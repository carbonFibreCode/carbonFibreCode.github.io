---
layout: default
title: Resume | Arun Kumar
---

<section class="section">
  <div class="section-header">
    <i class="fas fa-file-pdf"></i>
    <h2>Curriculum Vitae</h2>
    <a href="{{ '/assets/arunkumar-resume.pdf' | relative_url }}" class="btn btn-primary" download style="margin-left: auto;">
      <i class="fas fa-download"></i> Download PDF
    </a>
  </div>

  <div class="resume-viewer-container">
    <iframe 
      src="{{ '/assets/arunkumar-resume.pdf' | relative_url }}" 
      width="100%" 
      height="1000px" 
      style="border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-secondary);"
      title="Arun Kumar Resume"
    >
      <p>It appears your browser doesn't support embedded PDFs. 
         <a href="{{ '/assets/arunkumar-resume.pdf' | relative_url }}">Click here to download the PDF instead.</a>
      </p>
    </iframe>
  </div>
</section>
