let poData = []; // store parsed data lines

function generateDashboard(data) {
  poData = data; // store data

  // ... existing rendering logic ...

  dashboard.querySelectorAll('.approve').forEach((btn, index) => {
    btn.addEventListener('click', () => {
      // Update status in data
      poData[index].status = 'approved';

      // Update UI: change button and badge
      btn.textContent = "Approved";
      btn.className = "approve"; // style class for approved
      btn.disabled = true; // optionally disable after approval

      const badge = btn.closest('.po-card').querySelector('.badge');
      badge.textContent = "APPROVED";
      badge.className = "badge badge-clean";

      // Recalculate counts
      recalculateCounts();
    });
  });
}

function recalculateCounts() {
  const approvedCount = poData.filter(line => line.status === 'approved').length;
  const reviewCount = poData.filter(line => !line.status || line.status === 'review').length;
  const cleanCount = poData.filter(line => line.status === 'clean').length;

  document.getElementById('cleanCount').textContent = cleanCount;
  document.getElementById('reviewCount').textContent = reviewCount;
}
