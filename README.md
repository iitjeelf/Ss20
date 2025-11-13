    btn.onclick = () => {
      if(progress.answers[progress.currentIndex] === opt){
        progress.answers[progress.currentIndex] = null; // deselect if already selected
      } else {
        progress.answers[progress.currentIndex] = opt; // select new option
      }
      renderPalette();
      renderQuestion();
    };
    q.appendChild(btn);
  });

  // Update question number
  document.getElementById('qNumber').textContent = `Q${progress.currentIndex+1}/${examData.files.length}`;
}

// ==== NAVIGATION ====
document.getElementById('prevBtn').onclick = () => {
  if(progress.currentIndex > 0){
    progress.currentIndex--;
    renderQuestion();
    renderPalette();
    updateNavButtons();
  }
};
document.getElementById('nextBtn').onclick = () => {
  if(progress.currentIndex < examData.files.length - 1){
    progress.currentIndex++;
    renderQuestion();
    renderPalette();
    updateNavButtons();
  }
};
document.getElementById('markReviewBtn').onclick = () => {
  progress.review[progress.currentIndex] = !progress.review[progress.currentIndex];
  renderPalette();
};
document.getElementById('submitBtn').onclick = () => {
  showConfirm("Are you sure you want to SUBMIT the exam?", ok => {
    if(ok) submitExam();
  });
};

// ==== PALETTE ====
function renderPalette(){
  const container = document.getElementById('paletteContainer');
  container.innerHTML = '';
  progress.answers.forEach((ans, i) => {
    const btn = document.createElement('button');
    btn.textContent = i + 1;

    if(progress.review[i]){
      btn.classList.add('reviewed'); // yellow
      btn.classList.remove('selected');
    } else if(ans){
      btn.classList.add('selected'); // green
      btn.classList.remove('reviewed');
    } else {
      btn.classList.remove('selected', 'reviewed');
    }

    if(i === progress.currentIndex) btn.classList.add('current');

    btn.onclick = () => {
      progress.currentIndex = i;
      renderQuestion();
      renderPalette();
      updateNavButtons();
    };

    container.appendChild(btn);
  });
}

// ==== NAV BUTTON VISIBILITY ====
function updateNavButtons(){
  const prevBtn = document.getElementById('prevBtn');
  const nextBtn = document.getElementById('nextBtn');
  prevBtn.style.display = (progress.currentIndex === 0) ? 'none' : 'inline-block';
  nextBtn.style.display = (progress.currentIndex === examData.files.length - 1) ? 'none' : 'inline-block';
}

// ==== SUBMIT EXAM ====
function submitExam(){
  clearInterval(timerInterval);
  const subjData = {};
  examData.subjectNames.forEach((sub, i) => {
    if(!subjData[sub]) subjData[sub] = {correct:0, wrong:0, blank:0, total:0};
    const ans = progress.answers[i];
    if(ans === null) subjData[sub].blank++;
    else if(ans === examData.keys[i]) subjData[sub].correct++;
    else subjData[sub].wrong++;
    subjData[sub].total++;
  });

  let html = `<h2 style="color:#4B0082;">RESULT</h2>
  <div style="text-align:center;font-weight:bold;margin-bottom:10px;">
  Student: ${progress.student.name} | Section: ${progress.student.sec} | Roll: ${progress.student.roll} | Adm: ${progress.student.adm}
  </div>
  <table border="1" style="border-collapse:collapse;width:100%;margin:0 auto;text-align:center;">
  <tr style="background:#E6E6FA;">
    <th>Subject</th><th>Correct</th><th>Wrong</th><th>Blank</th><th>Total</th><th>%</th>
  </tr>`;

  let totalCorrect=0, totalWrong=0, totalBlank=0, totalQs=0;
  for(let sub in subjData){
    const s = subjData[sub];
    const perc = ((s.correct/s.total)*100).toFixed(2);
    totalCorrect += s.correct; totalWrong += s.wrong; totalBlank += s.blank; totalQs += s.total;
    html += `<tr style="background:#fff;">
      <td>${sub}</td>
      <td style="text-align:center;">${s.correct}</td>
      <td style="text-align:center;">${s.wrong}</td>
      <td style="text-align:center;">${s.blank}</td>
      <td style="text-align:center;">${s.total}</td>
      <td style="text-align:center;">${perc}%</td>
    </tr>`;
  }
  const totalPerc = ((totalCorrect/totalQs)*100).toFixed(2);
  html += `<tr style="background:#D8BFD8;font-weight:bold;">
    <td>Total</td>
    <td style="text-align:center;">${totalCorrect}</td>
    <td style="text-align:center;">${totalWrong}</td>
    <td style="text-align:center;">${totalBlank}</td>
    <td style="text-align:center;">${totalQs}</td>
    <td style="text-align:center;">${totalPerc}%</td>
  </tr></table>`;

  html += `<p style="text-align:center;margin-top:15px;color:#4B0082;font-weight:bold;">Thank you for attempting the exam!</p>`;
  html += `<div style="text-align:center;margin-top:10px;"><button onclick="logout()">Logout</button></div>`;
  document.body.innerHTML = html;
}

// ==== LOGOUT FUNCTION ====
function logout(){
  window.close(); // Try to close
  window.location.href = "about:blank"; // fallback
}

// ==== RED OVERLAY FOR BLOCKED ====
function showRedOverlay(){
  const overlay = document.getElementById('fullRedOverlay'); overlay.style.display='block';
  for(let i=0; i<7; i++){
    const s = document.createElement('div'); s.className='overlay-slice'; s.style.left=(i*100/7)+'%'; overlay.appendChild(s);
  }
  const accessBox = document.getElementById('accessDeniedBox'); accessBox.style.display='block';
  const pwdPrompt = () => {
    const pass = prompt("Enter password to bypass:");
    if(pass === 'lfjc2025'){ overlay.style.display='none'; accessBox.style.display='none'; }
  };
  document.getElementById('callAdminBtn').onclick = pwdPrompt;
}
</script>
</body>
</html>
