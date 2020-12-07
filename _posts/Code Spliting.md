









html2canvas / jspdf
```
import html2canvas from 'html2canvas';		
import jsPDF from 'jspdf';

 const printDocument = () => {
    const input = document.getElementById('main');
    if (!input) return;
    setTimeout(()=>setIsExporting(true),0);
    const h = input.scrollHeight;
    const w = input.scrollWidth;

    const options = {
      h,
      w,
      windowHeight: h,
    };
    html2canvas(input, options)
      .then(canvas => {
        const img = canvas.toDataURL('image/data');

        const pdf = new jsPDF('p', 'px', [w, h]);
        pdf.addImage(img, 'PNG', 0, 0, w, h);
        pdf.save('download.pdf');
      })
      .then(() => setIsExporting(false));
  };
```
```javscript
const printDocument = () => {
    const input = document.getElementById('main');
    if (!input) return;
    setIsExporting(true);
    const h = input.scrollHeight;
    const w = input.scrollWidth;

    const options = {
      h,
      w,
      windowHeight: h,
    };
    import('html2canvas')
      .then(({ default: html2canvas }) => html2canvas(input, options))
      .then(canvas => canvas.toDataURL('image/data'))
      .then(img => {
        import('jspdf').then(({ default: jsPDF }) => {
          const pdf = new jsPDF('p', 'px', [w, h]);
          pdf.addImage(img, 'PNG', 0, 0, w, h);
          pdf.save('download.pdf');
        });
      })
      .catch(() => {
        notification.error({
          message: 'Failed to Export. Please try again.',
        });
      })
      .finally(() => setIsExporting(false));
```