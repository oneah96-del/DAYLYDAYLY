LH 보증금 상호전환 계산기

<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>보증금 상호전환 계산기</title>
  <style>
    body {
      font-family: system-ui, -apple-system, 'Malgun Gothic', sans-serif;
      max-width: 600px;
      margin: 20px auto;
      padding: 20px;
      background: #fafafa;
    }
    h1 {
      font-size: 1.5rem;
      margin-bottom: 20px;
      text-align: center;
    }
    label {
      font-weight: 600;
      margin-top: 10px;
      display: block;
    }
    input, select {
      width: 100%;
      padding: 10px;
      margin-top: 5px;
      border-radius: 8px;
      border: 1px solid #ccc;
      font-size: 1rem;
    }
    button {
      margin-top: 20px;
      padding: 12px;
      width: 100%;
      background: #007bff;
      color: #fff;
      border: none;
      border-radius: 8px;
      font-size: 1rem;
      font-weight: 700;
      cursor: pointer;
    }
    button:hover {
      background: #0056b3;
    }
    .card {
      background: #fff;
      border-radius: 10px;
      padding: 20px;
      margin-top: 20px;
      box-shadow: 0 2px 5px rgba(0,0,0,0.05);
    }
    .summary {
      font-size: 1.1rem;
      font-weight: 600;
      margin-bottom: 15px;
    }
    .highlight {
      font-size: 1.2rem;
      font-weight: bold;
      color: #007bff;
      margin-bottom: 10px;
    }
    .warn {
      font-size: 0.95rem;
      color: #c00;
      margin-top: 15px;
      line-height: 1.5;
    }
    .footer {
      margin-top: 30px;
      font-size: 0.85rem;
      color: #777;
      text-align: center;
      line-height: 1.4;
    }
  </style>
</head>
<body>
  <h1>보증금 상호전환 계산기</h1>

  <label for="mode">계산 유형</label>
  <select id="mode">
    <option value="increase">보증금 증액</option>
    <option value="decrease">보증금 감액</option>
  </select>

  <label for="currentDeposit">기존 보증금 (원)</label>
  <input id="currentDeposit" type="text" value="10,000,000" />

  <label for="currentRent">기존 월임대료 (원)</label>
  <input id="currentRent" type="text" value="100,000" />

  <label for="amount">전환할 보증금 (원)</label>
  <input id="amount" type="text" value="2,000,000" />

  <label for="rate">전환 이율 (%)</label>
  <input id="rate" type="number" step="0.1" />

  <label for="maxRatio">최대 전환 비율 (%)</label>
  <input id="maxRatio" type="number" value="60" />
  <p class="note">기본 월임대료의 몇 %까지만 보증금으로 전환 가능</p>

  <label for="unit">전환 단위 (원)</label>
  <input id="unit" type="text" value="100,000" />
  <p class="note">전환 금액은 이 단위로 입력해야 합니다</p>

  <button id="calc">계산하기</button>

  <div id="output" class="card" style="display:none">
    <div class="summary" id="summaryText"></div>
    <div class="highlight" id="depositText"></div>
    <div class="highlight" id="rentText"></div>
    <div class="warn" id="warnText"></div>

    <div class="footer">
      <p class="disclaimer">※ 본 계산기는 참고용으로 제공되는 도구이며, 실제 계약에 대한 책임은 사용자에게 있습니다. 반드시 해당 기관의 공식 공고를 확인하시기 바랍니다.</p>
      <p class="copyright">© DAILYDAILY. All rights reserved.</p>
    </div>
  </div>

  <script>
    const addCommas = (num) => num.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ',');
    const removeCommas = (num) => num.replace(/,/g, '');

    window.addEventListener('DOMContentLoaded', () => {
      const mode = document.getElementById('mode').value;
      document.getElementById('rate').value = mode === 'increase' ? 7 : 3.5;
    });

    document.getElementById('mode').addEventListener('change', (e) => {
      document.getElementById('rate').value = e.target.value === 'increase' ? 7 : 3.5;
    });

    document.querySelectorAll('input[type="text"]').forEach(input => {
      input.addEventListener('input', (e) => {
        let value = removeCommas(e.target.value);
        if(!isNaN(value) && value !== '') {
          e.target.value = addCommas(value);
        }
      });
    });

    document.getElementById('calc').addEventListener('click', () => {
      const mode = document.getElementById('mode').value;
      const currentDeposit = Number(removeCommas(document.getElementById('currentDeposit').value)) || 0;
      const currentRent = Number(removeCommas(document.getElementById('currentRent').value)) || 0;
      const amount = Number(removeCommas(document.getElementById('amount').value)) || 0;
      const rate = Number(document.getElementById('rate').value) / 100;
      const maxRentRatio = Number(document.getElementById('maxRatio').value) / 100;
      const unit = Number(removeCommas(document.getElementById('unit').value)) || 100000;

      let newDeposit, newRent, summaryText = '', warnText = '';

      if (amount % unit !== 0) {
        warnText += `⚠️ 전환 금액은 ${addCommas(unit)}원 단위로 입력해주세요.\n`;
      }

      if (mode === 'increase') {
        const maxConvertibleRent = currentRent * maxRentRatio;
        const maxConvertibleDeposit = Math.floor(maxConvertibleRent * 12 / rate);
        const rentDecrease = Math.round(amount * rate / 12);
        newDeposit = currentDeposit + amount;
        newRent = currentRent - rentDecrease;

        summaryText = `월임대료가 ${addCommas(rentDecrease)}원 인하되어 ${addCommas(newRent)}원이 됩니다.`;
        document.getElementById('depositText').textContent = `변경 후 보증금: ${addCommas(newDeposit)}원`;
        document.getElementById('rentText').textContent = `변경 후 월임대료: ${addCommas(newRent)}원`;

        if (amount > maxConvertibleDeposit) {
          const excessDeposit = amount - maxConvertibleDeposit;
          const excessRent = Math.round(excessDeposit * rate / 12);
          warnText += `⚠️ 최대 전환 가능 보증금은 ${addCommas(maxConvertibleDeposit)}원입니다.\n`;
          warnText += `초과된 보증금 ${addCommas(excessDeposit)}원은 월임대료로 환산 시 ${addCommas(excessRent)}원에 해당합니다.`;
        }

      } else {
        const rentIncrease = Math.round(amount * rate / 12);
        newDeposit = currentDeposit - amount;
        newRent = currentRent + rentIncrease;

        summaryText = `월임대료가 ${addCommas(rentIncrease)}원 인상되어 ${addCommas(newRent)}원이 됩니다.`;
        document.getElementById('depositText').textContent = `변경 후 보증금: ${addCommas(newDeposit)}원`;
        document.getElementById('rentText').textContent = `변경 후 월임대료: ${addCommas(newRent)}원`;
      }

      document.getElementById('summaryText').textContent = summaryText;
      document.getElementById('warnText').textContent = warnText;
      document.getElementById('output').style.display = 'block';
    });
  </script>
</body>
</html>
