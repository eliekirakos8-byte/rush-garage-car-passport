
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Rush Garage | Car Passport</title>
  <style>
    * { box-sizing: border-box; }
    body {
      margin: 0;
      font-family: Arial, sans-serif;
      background: #101114;
      color: #f5f5f5;
    }
    header {
      text-align: center;
      padding: 35px 15px 25px;
      background: linear-gradient(135deg, #25272c, #101114);
      border-bottom: 3px solid #e63939;
    }
    .logo {
      font-size: 34px;
      font-weight: 900;
      letter-spacing: 1px;
    }
    .logo span { color: #ff4141; }
    header p { color: #c8c8c8; margin-bottom: 0; }
    main {
      width: min(92%, 600px);
      margin: 28px auto;
    }
    .panel {
      background: #1c1e23;
      border: 1px solid #34363d;
      border-radius: 16px;
      padding: 22px;
      margin-bottom: 20px;
    }
    h2 { margin-top: 0; }
    label { display: block; margin: 15px 0 8px; }
    input {
      width: 100%;
      padding: 14px;
      border-radius: 9px;
      border: 1px solid #555;
      background: #111216;
      color: white;
      font-size: 16px;
    }
    button {
      width: 100%;
      padding: 14px;
      margin-top: 13px;
      border: 0;
      border-radius: 9px;
      background: #e63939;
      color: white;
      font-size: 16px;
      font-weight: bold;
      cursor: pointer;
    }
    button:disabled { opacity: .6; }
    .language {
      width: auto;
      padding: 8px 14px;
      margin: 12px auto 0;
      background: #35373e;
    }
    .message { color: #ffaaaa; line-height: 1.6; }
    .car-title { color: #ff5555; }
    .record {
      padding: 14px 0;
      border-bottom: 1px solid #3b3d43;
      line-height: 1.8;
    }
    .record:last-child { border-bottom: 0; }
    .muted { color: #bcbec5; }
    footer {
      text-align: center;
      padding: 20px;
      color: #a8a8a8;
      font-size: 13px;
    }
  </style>
</head>
<body>
  <header>
    <div class="logo">RUSH <span>GARAGE</span></div>
    <p id="tagline">جواز السيارة الإلكتروني وسجل الصيانة</p>
    <button class="language" id="langButton" onclick="toggleLanguage()">
      English
    </button>
  </header>

  <main>
    <section class="panel">
      <h2 id="title">افحص سجل سيارتك</h2>
      <p id="intro" class="muted">
        أدخل الكود الموجود على جواز السيارة لمعرفة سجل الصيانة.
      </p>

      <form id="lookupForm">
        <label id="codeLabel" for="accessCode">كود السيارة</label>
        <input id="accessCode" type="text"
          placeholder="مثال: RG-..." required minlength="12"
          autocomplete="off">
        <button id="searchButton" type="submit">عرض سجل السيارة</button>
      </form>
      <p id="message" class="message" role="status"></p>
    </section>

    <section id="results" class="panel" hidden>
      <h2 id="carHeading">معلومات السيارة</h2>
      <div id="carInfo"></div>
      <h2 id="historyHeading" style="margin-top:25px">
        سجل الصيانة
      </h2>
      <div id="records"></div>
    </section>
  </main>

  <footer>
    Rush Garage · Car Passport
  </footer>

  <script>
    // استبدل القيمتين بمعلومات مشروعك من Supabase.
    const SUPABASE_URL = "Project URL";
    const SUPABASE_KEY = "Publishable key";

    let english = false;

    function toggleLanguage() {
      english = !english;
      document.documentElement.lang = english ? "en" : "ar";
      document.documentElement.dir = english ? "ltr" : "rtl";

      document.getElementById("tagline").textContent =
        english ? "Your digital car service history" :
        "جواز السيارة الإلكتروني وسجل الصيانة";
      document.getElementById("langButton").textContent =
        english ? "العربية" : "English";
      document.getElementById("title").textContent =
        english ? "Check your car history" : "افحص سجل سيارتك";
      document.getElementById("intro").textContent =
        english ? "Enter your car passport code to view service history." :
        "أدخل الكود الموجود على جواز السيارة لمعرفة سجل الصيانة.";
      document.getElementById("codeLabel").textContent =
        english ? "Car access code" : "كود السيارة";
      document.getElementById("searchButton").textContent =
        english ? "View car history" : "عرض سجل السيارة";
      document.getElementById("carHeading").textContent =
        english ? "Car information" : "معلومات السيارة";
      document.getElementById("historyHeading").textContent =
        english ? "Service history" : "سجل الصيانة";
    }

    document.getElementById("lookupForm").addEventListener("submit",
      async function(event) {
        event.preventDefault();

        const code = document.getElementById("accessCode").value.trim();
        const message = document.getElementById("message");
        const button = document.getElementById("searchButton");
        const results = document.getElementById("results");

        results.hidden = true;
        message.textContent = "";

        if (SUPABASE_URL.includes("_HERE") ||
            SUPABASE_KEY.includes("_HERE")) {
          message.textContent = english
            ? "Setup needed: add your Supabase project URL and publishable key in the code."
            : "بعد في خطوة إعداد: حط رابط مشروع Supabase والمفتاح العام بمكانهما بالكود.";
          return;
        }

        button.disabled = true;
        button.textContent = english ? "Searching..." : "عم نفتّش...";

        try {
          const response = await fetch(
            SUPABASE_URL.replace(/\/$/, "") +
              "/rest/v1/rpc/lookup_car_history",
            {
              method: "POST",
              headers: {
                "Content-Type": "application/json",
                "apikey": SUPABASE_KEY,
                "Authorization": "Bearer " + SUPABASE_KEY
              },
              body: JSON.stringify({ p_access_code: code })
            }
          );

          if (!response.ok) {
            throw new Error("Request failed");
          }

          const data = await response.json();

          if (!data || !data.car) {
            message.textContent = english
              ? "No car found. Check the code and try again."
              : "ما لقينا سيارة بهالكود. تأكد من الكود وجرّب مرة تانية.";
            return;
          }

          const carInfo = document.getElementById("carInfo");
          carInfo.replaceChildren();

          const car = data.car;
          const details = [
            [english ? "Make" : "الشركة", car.make],
            [english ? "Model" : "الموديل", car.model],
            [english ? "Year" : "سنة الصنع", car.year]
          ];

          details.forEach(function(item) {
            if (item[1] !== null && item[1] !== undefined &&
                item[1] !== "") {
              const p = document.createElement("p");
              p.textContent = item[0] + ": " + item[1];
              carInfo.appendChild(p);
            }
          });

          const records = document.getElementById("records");
          records.replaceChildren();

          if (!data.service_records ||
              data.service_records.length === 0) {
            records.textContent = english
              ? "No service records yet."
              : "ما في سجلات صيانة مسجّلة لهالسيارة بعد.";
          } else {
            data.service_records.forEach(function(record) {
              const article = document.createElement("article");
              article.className = "record";

              const heading = document.createElement("strong");
              heading.className = "car-title";
              heading.textContent =
                (record.service_type || (english ? "Service" : "صيانة")) +
                (record.service_date ? " · " + record.service_date : "");
              article.appendChild(heading);

              [
                [english ? "Description" : "الوصف", record.description],
                [english ? "Parts used" : "قطع مستعملة", record.parts_used],
                [english ? "Labor cost" : "أجرة اليد العاملة", record.labor_cost],
                [english ? "Parts cost" : "كلفة القطع", record.parts_cost]
              ].forEach(function(item) {
                if (item[1] !== null && item[1] !== undefined &&
                    item[1] !== "") {
                  const p = document.createElement("p");
                  p.textContent = item[0] + ": " + item[1];
                  article.appendChild(p);
                }
              });

              records.appendChild(article);
            });
          }

          results.hidden = false;
        } catch (error) {
          message.textContent = english
            ? "Could not connect. Check your Supabase setup and try again."
            : "ما قدرنا نتصل بقاعدة البيانات. تأكد من إعدادات Supabase.";
        } finally {
          button.disabled = false;
          button.textContent = english
            ? "View car history" : "عرض سجل السيارة";
        }
      }
    );
  </script>
</body>
</html>
