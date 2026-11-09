<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<title>Журнал охорони</title>

<style>
* { box-sizing: border-box; }

body {
  margin: 0;
  font-family: Arial, sans-serif;
  background: #f1f5f9;
  color: #111827;
}

header {
  background: #111827;
  color: white;
  text-align: center;
  padding: 22px 15px;
}

header h1 {
  margin: 0 0 6px;
  font-size: 24px;
}

header p {
  margin: 0;
  opacity: 0.8;
}

.container {
  max-width: 600px;
  margin: auto;
  padding: 15px;
}

.card {
  background: white;
  border-radius: 16px;
  padding: 18px;
  margin-bottom: 15px;
  box-shadow: 0 4px 15px rgba(0,0,0,0.08);
}

h2 {
  margin-top: 0;
}

label {
  display: block;
  font-weight: bold;
  margin-top: 14px;
  margin-bottom: 6px;
}

input {
  width: 100%;
  padding: 14px;
  border: 1px solid #cbd5e1;
  border-radius: 10px;
  font-size: 16px;
}

button {
  width: 100%;
  margin-top: 16px;
  padding: 15px;
  border: none;
  border-radius: 10px;
  background: #2563eb;
  color: white;
  font-size: 17px;
  font-weight: bold;
}

.save-btn:active,
.delete-btn:active {
  transform: scale(0.98);
}

.search {
  margin-bottom: 15px;
}

.record {
  padding: 15px 0;
  border-bottom: 1px solid #e5e7eb;
}

.record:last-child {
  border-bottom: none;
}

.car {
  font-size: 19px;
  font-weight: bold;
}

.time {
  color: #64748b;
  font-size: 13px;
  margin-top: 7px;
}

.photo-preview,
.record img {
  width: 100%;
  margin-top: 12px;
  border-radius: 12px;
  max-height: 300px;
  object-fit: cover;
}

.photo-preview {
  display: none;
}

.delete-btn {
  background: #dc2626;
  margin-top: 12px;
  padding: 11px;
  font-size: 15px;
}

.empty {
  text-align: center;
  color: #64748b;
  padding: 20px;
}
</style>
</head>

<body>

<header>
  <h1>🛡️ Журнал охорони</h1>
  <p>Швидкий запис автомобілів</p>
</header>

<div class="container">

  <div class="card">

    <label>🚗 Номер автомобіля</label>
    <input id="carNumber" placeholder="AA 1234 BB">

    <label>📞 Номер телефону</label>
    <input id="phone" value="+380" inputmode="tel">

    <label>👤 Ім'я</label>
    <input id="name" placeholder="Ім'я водія">

    <label>📸 Фото автомобіля</label>
    <input id="photo" type="file" accept="image/*" capture="environment">

    <img id="preview" class="photo-preview">

    <button class="save-btn" onclick="saveRecord()">
      💾 Зберегти запис
    </button>

  </div>

  <div class="card">

    <h2>📋 Записи</h2>

    <input
      class="search"
      id="search"
      placeholder="🔎 Пошук: номер, телефон або ім'я"
      oninput="renderRecords()"
    >

    <div id="records"></div>

  </div>

</div>

<script>

let photoData = "";

const photoInput = document.getElementById("photo");
const preview = document.getElementById("preview");

photoInput.addEventListener("change", function(event) {

  const file = event.target.files[0];

  if (!file) return;

  const reader = new FileReader();

  reader.onload = function(e) {
    photoData = e.target.result;
    preview.src = photoData;
    preview.style.display = "block";
  };

  reader.readAsDataURL(file);

});


function saveRecord() {

  const carNumber =
    document.getElementById("carNumber").value.trim();

  const phone =
    document.getElementById("phone").value.trim();

  const name =
    document.getElementById("name").value.trim();


  if (!carNumber) {
    alert("Введіть номер автомобіля!");
    return;
  }


  const records =
    JSON.parse(localStorage.getItem("guardRecords") || "[]");


  records.unshift({
    carNumber: carNumber,
    phone: phone,
    name: name,
    time: new Date().toLocaleString("uk-UA"),
    photo: photoData
  });


  localStorage.setItem(
    "guardRecords",
    JSON.stringify(records)
  );


  document.getElementById("carNumber").value = "";
  document.getElementById("phone").value = "+380";
  document.getElementById("name").value = "";
  document.getElementById("photo").value = "";

  preview.style.display = "none";
  photoData = "";

  renderRecords();

}


function deleteRecord(index) {

  if (!confirm("Видалити цей запис?")) return;

  const records =
    JSON.parse(localStorage.getItem("guardRecords") || "[]");

  records.splice(index, 1);

  localStorage.setItem(
    "guardRecords",
    JSON.stringify(records)
  );

  renderRecords();

}


function renderRecords() {

  const records =
    JSON.parse(localStorage.getItem("guardRecords") || "[]");

  const search =
    document.getElementById("search").value
      .toLowerCase()
      .trim();

  const container =
    document.getElementById("records");


  const filtered = records
    .map(function(record, index) {
      return { ...record, originalIndex: index };
    })
    .filter(function(record) {

      const text = (
        record.carNumber + " " +
        record.phone + " " +
        record.name
      ).toLowerCase();

      return text.includes(search);

    });


  if (filtered.length === 0) {

    container.innerHTML =
      '<div class="empty">Записів не знайдено</div>';

    return;
  }


  container.innerHTML =
    filtered.map(function(record) {

      return `
        <div class="record">

          <div class="car">
            🚗 ${record.carNumber}
          </div>

          <div>👤 ${record.name || "—"}</div>

          <div>📞 ${record.phone || "—"}</div>

          <div class="time">
            🕒 ${record.time}
          </div>

          ${
            record.photo
              ? `<img src="${record.photo}">`
              : ""
          }

          <button
            class="delete-btn"
            onclick="deleteRecord(${record.originalIndex})"
          >
            🗑️ Видалити запис
          </button>

        </div>
      `;

    }).join("");

}


renderRecords();

</script>

</body>
</html> guard-log
