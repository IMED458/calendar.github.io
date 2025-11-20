<html lang="ka">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>მორიგე ექიმების კალენდარი</title>
  <script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.13.1/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.13.1/firebase-auth-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.13.1/firebase-database-compat.js"></script>
  <link href="https://fonts.googleapis.com/css2?family=BPG+Nino+Mtavruli:wght@400;600&display=swap" rel="stylesheet">
  <style>
    :root {
      --primary: #1e40af;
      --primary-dark: #1e3a8a;
      --accent: #10b981;
      --light: #f8fafc;
      --gray: #e2e8f0;
      --text: #1e293b;
      --border: #cbd5e1;
      --danger: #ef4444;
      --today-color: #dc2626;
      --weekend: #e5e7eb;
    }
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body { font-family: 'BPG Nino Mtavruli', sans-serif; background: linear-gradient(135deg, #e0e7ff 0%, #c7d2fe 100%); color: var(--text); min-height: 100vh; padding: 15px; }
    .container { max-width: 1400px; margin: 0 auto; }
    .header { background: white; padding: 18px; border-radius: 14px; box-shadow: 0 4px 16px rgba(0,0,0,0.1); text-align: center; margin-bottom: 20px; position: relative; }
    .header h1 { font-size: 24px; color: var(--primary-dark); margin-bottom: 6px; }
    .add-shift-btn { position: absolute; top: 15px; right: 15px; background: var(--primary); color: white; border: none; width: 48px; height: 48px; border-radius: 50%; font-size: 24px; font-weight: bold; cursor: pointer; box-shadow: 0 4px 12px rgba(30,64,175,0.3); }
    .add-shift-btn:hover { background: var(--primary-dark); transform: scale(1.1); }
    .calendar-section { background: white; padding: 20px; border-radius: 14px; box-shadow: 0 4px 16px rgba(0,0,0,0.1); }
    .calendar-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 18px; flex-wrap: wrap; gap: 10px; }
    .calendar-header h2 { font-size: 20px; color: var(--primary-dark); }
    .nav-btn { background: var(--light); border: 2px solid var(--gray); padding: 8px 12px; border-radius: 8px; cursor: pointer; font-weight: 600; font-size: 14px; }
    .calendar-grid { display: grid; grid-template-columns: repeat(7, 1fr); gap: 6px; }
    .day-name { text-align: center; font-weight: 600; padding: 10px; background: var(--primary); color: white; border-radius: 8px; font-size: 13px; }
    .day-cell { min-height: 50px; border: 2px solid var(--gray); border-radius: 10px; padding: 6px; font-size: 13px; cursor: pointer; background: #fafafa; transition: 0.2s; position: relative; }
    .day-cell:hover { background: #eff6ff; border-color: var(--primary); }
    .day-cell.today { background: #fee2e2 !important; border-color: var(--today-color) !important; color: var(--today-color) !important; font-weight: 700; }
    .day-cell.has-shift { background: #ecfdf5; border-color: var(--accent); }
    .day-cell .date-num { font-weight: 600; margin-bottom: 3px; }
    .day-cell .shift-count { font-size: 10px; color: var(--accent); font-weight: bold; }
    .dept-search { margin: 15px 0; padding: 10px; border: 2px solid var(--gray); border-radius: 10px; font-size: 14px; width: 100%; }
    .departments-grid { display: grid; gap: 14px; margin-top: 18px; }
    .dept-card { border: 2px solid var(--border); border-radius: 12px; overflow: hidden; box-shadow: 0 2px 8px rgba(0,0,0,0.05); }
    .dept-header { background: var(--primary-dark); color: white; padding: 12px 16px; font-weight: 600; font-size: 15px; }
    .shift-item { padding: 10px 16px; border-bottom: 1px solid var(--gray); display: flex; justify-content: space-between; align-items: center; font-size: 13px; }
    .shift-item:last-child { border-bottom: none; }
    .shift-hours { background: #ecfdf5; color: #059669; padding: 3px 8px; border-radius: 6px; font-weight: 600; font-size: 11px; }
    .delete-btn { background: var(--danger); color: white; border: none; padding: 3px 7px; border-radius: 5px; font-size: 10px; cursor: pointer; }
    .modal { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.5); z-index: 1000; justify-content: center; align-items: center; padding: 15px; }
    .modal.active { display: flex; }
    .modal-content { background: white; border-radius: 16px; width: 100%; max-width: 480px; max-height: 90vh; overflow-y: auto; box-shadow: 0 10px 30px rgba(0,0,0,0.2); }
    .modal-header { padding: 18px; border-bottom: 1px solid var(--gray); text-align: center; font-size: 18px; font-weight: 600; color: var(--primary-dark); }
    .modal-body { padding: 18px; }
    .form-group { margin-bottom: 16px; }
    .form-group label { display: block; margin-bottom: 6px; font-weight: 500; font-size: 14px; }
    .form-group input, .form-group select { width: 100%; padding: 11px; border: 2px solid var(--gray); border-radius: 10px; font-size: 14px; }
    .search-input { padding: 11px; border: 2px solid var(--gray); border-radius: 10px; font-size: 14px; margin-bottom: 10px; }
    .doctor-list { max-height: 300px; overflow-y: auto; border: 1px solid var(--gray); border-radius: 8px; margin-bottom: 16px; background: white; }
    .doctor-item { padding: 10px 12px; border-bottom: 1px solid #eee; cursor: pointer; font-size: 13px; transition: 0.2s; }
    .doctor-item:hover { background: #f0f9ff; }
    .doctor-item.selected { background: #dbeafe; font-weight: 600; }
    .btn { padding: 11px 18px; border: none; border-radius: 10px; font-weight: 600; cursor: pointer; font-size: 14px; width: 100%; margin-top: 10px; }
    .btn-primary { background: var(--primary); color: white; }
    .btn-accent { background: var(--accent); color: white; }
    .btn-secondary { background: #6b7280; color: white; }
    .export-btn { background: #7c3aed; color: white; padding: 12px 20px; border-radius: 12px; font-weight: 600; cursor: pointer; font-size: 15px; margin-top: 20px; border: none; }
    .status { position: fixed; top: 15px; left: 15px; background: #f59e0b; color: white; padding: 8px 14px; border-radius: 8px; font-size: 13px; z-index: 999; box-shadow: 0 2px 8px rgba(0,0,0,0.2); }
  </style>
</head>
<body>
  <div id="status" class="status">იტვირთება...</div>
  <div class="container">
    <div class="header">
      <h1>მორიგე ექიმების კალენდარი</h1>
      <p>დააჭირეთ თარიღს ან + ღილაკს</p>
      <button class="add-shift-btn" id="open-modal-btn">+</button>
    </div>
    <div class="calendar-section">
      <div class="calendar-header">
        <h2 id="month-year"></h2>
        <div>
          <button class="nav-btn" id="prev-month">წინა</button>
          <button class="nav-btn" id="today-btn">დღეს</button>
          <button class="nav-btn" id="next-month">შემდეგი</button>
        </div>
      </div>
      <div id="calendar-grid" class="calendar-grid">
        <div class="day-name">ორშ</div><div class="day-name">სამ</div><div class="day-name">ოთხ</div><div class="day-name">ხუთ</div><div class="day-name">პარ</div><div class="day-name">შაბ</div><div class="day-name">კვი</div>
      </div>
      <div id="selected-date-view" style="display: none; margin-top: 20px;">
        <h3 style="margin: 15px 0; color: var(--primary-dark);" id="selected-date-title"></h3>
        <input type="text" id="dept-search" class="dept-search" placeholder="ძებნა განყოფილებაში..." />
        <div id="departments-grid" class="departments-grid"></div>
        <button class="export-btn" id="export-excel">Excel ექსპორტი (განყოფილებებით)</button>
      </div>
    </div>
  </div>

  <!-- Modal -->
  <div class="modal" id="shift-modal">
    <div class="modal-content">
      <div class="modal-header">მორიგეობის დამატება</div>
      <div class="modal-body">
        <div class="form-group"><label>სპეციალობა</label><select id="specialty-filter"><option value="">ყველა</option></select></div>
        <div class="form-group"><label>ექიმის ძებნა</label><input type="text" id="doctor-search" class="search-input" placeholder="სახელი ან ტელეფონი..." /></div>
        <div class="doctor-list" id="doctor-list"></div>
        <div class="form-group"><label>ტელეფონი</label><input type="tel" id="modal-phone" readonly /></div>
        <div class="form-group"><label>თარიღი</label><input type="date" id="modal-date" required /></div>
        <div class="form-group"><label>მორიგეობა</label>
          <select id="modal-hours" required>
            <option value="">აირჩიეთ...</option>
            <option value="8">8 საათი</option>
            <option value="12">12 საათი</option>
            <option value="16">16 საათი</option>
            <option value="24">24 საათი</option>
          </select>
        </div>
        <div class="repeat-section">
          <label>გამეორება</label>
          <select id="repeat-type">
            <option value="none">არ განმეორდეს</option>
            <option value="daily">ყოველ დღე</option>
            <option value="every2">ყოველ 2 დღეში</option>
            <option value="every4">ყოველ 4 დღეში</option>
          </select>
          <input type="number" id="repeat-until" placeholder="დღეების რაოდენობა" style="margin-top:8px;display:none;" min="1"/>
        </div>
        <button class="btn btn-primary" id="add-shift-final">დამატება</button>

        <div style="margin-top:30px;padding:20px;background:#f0f9ff;border-radius:12px;border:2px dashed #3b82f6;">
          <h4 style="margin-bottom:15px;color:var(--primary-dark);">ახალი ექიმის დამატება</h4>
          <div class="form-group"><input type="text" id="new-name" placeholder="სახელი გვარი" /></div>
          <div class="form-group">
            <select id="new-specialty-select">
              <option value="">აირჩიეთ განყოფილება</option>
              <option>CT ოპერატორი</option><option>CT რადიოლოგი</option><option>ანგიოქირურგია</option><option>გადაუდებელი მედიცინა</option><option>გინეკოლოგია</option><option>ენდოსკოპია</option><option>ექოსკოპია</option><option>ზოგადი რეანიმაცია</option><option>ზოგადი ქირურგია</option><option>თორაკო ქირურგია</option><option>ინფექციური სნეულებები</option><option>კარდიოლოგია</option><option>ლაბორატორია</option><option>ნევროლოგია</option><option>ნეირო ქირურგია</option><option>ნეფროლოგია</option><option>პულმონოლოგია</option><option>X-ray რადიოლოგი</option><option>რენტგენი</option><option>ტრავმატოლოგია</option><option>უროლოგია</option><option>ყბა–სახის ქირურგია</option><option>შინაგანი მედიცინა</option><option>ნეირო რეანიმაცია</option><option>კარდიო რეანიმაცია</option><option>ბავშვთა რეანიმაცია</option><option>ბავშვთა გადაუდებელი მედიცინა</option><option>ბავშვთა ონკო-ჰემატოლოგია</option><option>ბავშვთა ქირურგია</option><option>პედიატრია</option><option>ანესთეზია</option><option>ჰემატოლოგია 3</option><option>ჰემატოლოგია 10</option><option>ჰეპატოლოგია</option>
            </select>
          </div>
          <div class="form-group"><input type="tel" id="new-phone" placeholder="ტელეფონი" /></div>
          <button class="btn btn-accent" id="add-new-doctor">ექიმის დამატება</button>
        </div>
        <button class="btn btn-secondary" id="close-modal" style="margin-top:12px;">დახურვა</button>
      </div>
    </div>
  </div>

  <script>
    // Firebase
    const firebaseConfig = { apiKey: "AIzaSyAH2CvRxLYqd3KGAsRoTvzCTH4x8bZNnl0", authDomain: "doctor-calendar-db.firebaseapp.com", databaseURL: "https://doctor-calendar-db-default-rtdb.firebaseio.com", projectId: "doctor-calendar-db", storageBucket: "doctor-calendar-db.firebasestorage.app", messagingSenderId: "1085600886719", appId: "1:1085600886719:web:7e22b240cbea045a443b0a" };
    firebase.initializeApp(firebaseConfig);
    firebase.auth().signInAnonymously();
    const db = firebase.database();
    const shiftsRef = db.ref('shifts');
    const doctorsRef = db.ref('doctors');

    let allShifts = [], doctors = [], currentMonth = new Date().getMonth(), currentYear = new Date().getFullYear(), selectedDate = null, selectedDoctor = null;

    const defaultDoctors = [
      {name:"პაატა ბარათაშვილი",specialty:"X-ray რადიოლოგი",phone:"593 311 748"},{name:"ვაჟა თავბერიძე",specialty:"X-ray რადიოლოგი",phone:"551 470 471"},
      {name:"მაია",specialty:"რენტგენი",phone:"557 654 351"},{name:"ნინო",specialty:"რენტგენი",phone:"599 400 311"},{name:"ნაზი",specialty:"რენტგენი",phone:"555 181 801"},
      {name:"ნინო კიკვაძე",specialty:"გადაუდებელი მედიცინა",phone:"598 739 756"},{name:"ანა დალაქიშვილი",specialty:"გადაუდებელი მედიცინა",phone:"555 606 064"},
      {name:"მარიამი",specialty:"რენტგენი",phone:"598 100 644"},{name:"ნიკა მაჩაიძე",specialty:"CT ოპერატორი",phone:"598 295 798"},{name:"მარიამი",specialty:"CT ოპერატორი",phone:"599 216 624"},
      {name:"ზურა ქოჩერაშვილი",specialty:"CT ოპერატორი",phone:"557 767 362"},{name:"მაია დემურიშვილი",specialty:"CT რადიოლოგი",phone:"555 258 800"},
      {name:"ვალერიანე უხურგუნაშვილი",specialty:"CT რადიოლოგი",phone:"558 333 455"},{name:"ცისია კახაძე",specialty:"CT რადიოლოგი",phone:"599 407 560"},
      {name:"ჯუბა ნაზარაშვილი",specialty:"CT ოპერატორი",phone:"571 036 317"},{name:"მანანა გოგოლაძე",specialty:"ექოსკოპია",phone:"577 450 049"},
      {name:"ანა ინგოროყვა",specialty:"ექოსკოპია",phone:"599 222 201"},{name:"მარიამ გავაშელი",specialty:"ექოსკოპია",phone:"544 447 346"},
      {name:"თამარ გოგელია",specialty:"ექოსკოპია",phone:"557 424 363"},{name:"ირინა მოდებაძე",specialty:"ექოსკოპია",phone:"577 090 967"},
      {name:"ლაბორატორია",specialty:"ლაბორატორია",phone:"577 101 949"},{name:"ირაკლი დევიძე",specialty:"ყბა–სახის ქირურგია",phone:"597 03 05 40"},
      {name:"გიორგი გვენეტაძე",specialty:"ყბა–სახის ქირურგია",phone:"599 62 99 91"},{name:"ერეკლე გელაშვილი",specialty:"ყბა–სახის ქირურგია",phone:"597 02 20 99"},
      {name:"ნუნუკა გურაბანიძე",specialty:"ყბა–სახის ქირურგია",phone:"551 159 797"},{name:"გრიგოლ ჯავახაძე",specialty:"ყბა–სახის ქირურგია",phone:"597 098 116"},
      {name:"შალვა ჭოველიძე",specialty:"უროლოგია",phone:"577 460 025"},{name:"ნიკოლოზ გვარამია",specialty:"უროლოგია",phone:"597 774 091"},
      {name:"ვუგარ სადიკოვი",specialty:"უროლოგია",phone:"557 175 005"},{name:"ნანა გოგოხია",specialty:"უროლოგია",phone:"557 497 474"},
      {name:"მარიკა ყურაშვილი",specialty:"უროლოგია",phone:"555 213 650"},{name:"ზაური თაქთაქიშილი",specialty:"უროლოგია",phone:"551 591 774"},
      {name:"გიგი ორაგველიძე",specialty:"უროლოგია",phone:"511 282 879"},{name:"გიორგი ხიზანიშვილი",specialty:"ტრავმატოლოგია",phone:"595 914 096"},
      {name:"კახა გოშაძე",specialty:"ტრავმატოლოგია",phone:"598 787 859"},{name:"ზურა ჩხარტიშვილი",specialty:"ტრავმატოლოგია",phone:"599 055 181"},
      {name:"ნიკა ლომიძე",specialty:"ტრავმატოლოგია",phone:"599 808 191"},{name:"ნიკა რაზმაძე",specialty:"ტრავმატოლოგია",phone:"579 775 674"},
      {name:"გურამ ჩაჩუა",specialty:"ნეირო ქირურგია",phone:"579 031 178"},{name:"მიხეილ გურასპიშვილი",specialty:"ნეირო ქირურგია",phone:"555 191 378"},
      {name:"ოთარ გახოკია",specialty:"ნეირო ქირურგია",phone:"558 344 233"},{name:"არჩილ წიკლაური",specialty:"ნეირო ქირურგია",phone:"558 566 848"},
      {name:"ლუკა ლეკაშვილი",specialty:"ნეირო ქირურგია",phone:"595 455 135"},{name:"ლუკა გოგოტიშვილი",specialty:"ნეირო ქირურგია",phone:"592 861 741"},
      {name:"კორპორატიული",specialty:"ნეირო ქირურგია",phone:"511 453 571"},{name:"ნეირორეანიმაცია",specialty:"ნეირო რეანიმაცია",phone:"511 453 576"},
      {name:"ნინო ხარაიშვილი",specialty:"ნევროლოგია",phone:"593 151 588"},{name:"ნათია ხაჩიძე",specialty:"ნევროლოგია",phone:"598 61 06 24"},
      {name:"ალექსი მაღლაკელიძე",specialty:"ნევროლოგია",phone:"591 06 52 37"},{name:"თამთა კარანაძე",specialty:"ნევროლოგია",phone:"577 395 080"},
      {name:"ჟანა",specialty:"ნევროლოგია",phone:"579 379 252"},{name:"ქრისტინე დვალაძე",specialty:"ნევროლოგია",phone:"568 03 03 36"},
      {name:"ნათია კურტანიძე",specialty:"ნევროლოგია",phone:"599 70 57 33"},{name:"ანა შუბითიძე",specialty:"ნევროლოგია",phone:"555 37 59 68"},
      {name:"ანა ქურხული",specialty:"ნევროლოგია",phone:"568 908 466"},{name:"ირინა ჯაჯანიძე",specialty:"გინეკოლოგია",phone:"599 90 14 58"},
      {name:"რუსუდან ფიცხელაური",specialty:"გინეკოლოგია",phone:"599 67 61 40"},{name:"ნინო ხათრიძე",specialty:"გინეკოლოგია",phone:"598 48 21 42"},
      {name:"დიანა მირზაშვილი",specialty:"გინეკოლოგია",phone:"599 90 42 98"},{name:"თინა ჩალიგავა",specialty:"გინეკოლოგია",phone:"599 13 07 08"},
      {name:"ნინო შარაშენიძე",specialty:"ჰემატოლოგია 3",phone:"599 91 49 91"},{name:"ია მალაშხია",specialty:"ჰემატოლოგია 3",phone:"599 490 305"},
      {name:"შამო მუსაევი",specialty:"ჰემატოლოგია 3",phone:"557 949 226"},{name:"თაკო აზიკური",specialty:"ჰემატოლოგია 3",phone:"593 545 233"},
      {name:"ნატალია ნადიკაშვილი",specialty:"ჰემატოლოგია 3",phone:"577 222 970"},{name:"ჯონდი ჭავჭანიძე",specialty:"ენდოსკოპია",phone:"577 453 405"},
      {name:"დავით გობეჯიშვილი",specialty:"ენდოსკოპია",phone:"599 933 584"},{name:"თეიმურაზ სამადაშვილი",specialty:"ენდოსკოპია",phone:"598 22 22 46"},
      {name:"ირაკლი შეკლაშვილი",specialty:"ენდოსკოპია",phone:"577 339 956"},{name:"მარიკა წერეთელი",specialty:"ინფექციური სნეულებები",phone:"593 362 987"},
      {name:"ნუცა დონაძე",specialty:"ინფექციური სნეულებები",phone:"599 89 08 29"},{name:"თაკო ზაზაძე",specialty:"ინფექციური სნეულებები",phone:"597 777 113"},
      {name:"თამარ წერეთელი",specialty:"ინფექციური სნეულებები",phone:"555 558 333"},{name:"ია ბაღაშვილი",specialty:"ინფექციური სნეულებები",phone:"577 58 82 05"},
      {name:"ირმა მარკოიძე",specialty:"ინფექციური სნეულებები",phone:"599 470 228"},{name:"ციცი მაღლაფერიძე",specialty:"ინფექციური სნეულებები",phone:"579 70 60 81"},
      {name:"ნინო წურწუმია",specialty:"ინფექციური სნეულებები",phone:"557 58 78 34"},{name:"ნინო აბულაძე",specialty:"ინფექციური სნეულებები",phone:"599 060 194"},
      {name:"თამარი ტურიაშვილი",specialty:"ინფექციური სნეულებები",phone:"598 005 186"},{name:"ირინა კილაძე",specialty:"ინფექციური სნეულებები",phone:"599 88 35 77"},
      {name:"ეკატერინე მარკოზია",specialty:"ინფექციური სნეულებები",phone:"555 739 633"},{name:"ლაშა სარალიღე",specialty:"ზოგადი ქირურგია",phone:"599 977 762"},
      {name:"მაია ლობჟანიძე",specialty:"ზოგადი ქირურგია",phone:"577 671 710"},{name:"დავით ვარდოსანიძე",specialty:"ზოგადი ქირურგია",phone:"577 671 705"},
      {name:"ზაზა მანელიძე",specialty:"ზოგადი ქირურგია",phone:"595 582 876"},{name:"ირაკლი კაჭახიძე",specialty:"ზოგადი ქირურგია",phone:"577 671 707"},
      {name:"ლალი ახმეტელი",specialty:"ზოგადი ქირურგია",phone:"577 553 311"},{name:"ლია საგინაშვილი",specialty:"ზოგადი ქირურგია",phone:"599 503 567"},
      {name:"ბესო ირემაშვილი",specialty:"ზოგადი ქირურგია",phone:"595 300 719"},{name:"ონისე ტყეშელაშვილი",specialty:"ზოგადი ქირურგია",phone:"574 219 219"},
      {name:"გიორგი შუბითიძე",specialty:"ზოგადი ქირურგია",phone:"595 418 040"},{name:"გუგა ზაალიშვილი",specialty:"თორაკო ქირურგია",phone:"577 459 556"},
      {name:"რობერტი გობეჩია",specialty:"თორაკო ქირურგია",phone:"599 931 120"},{name:"ვასო ბაბიაშვილი",specialty:"თორაკო ქირურგია",phone:"557 752 565"},
      {name:"დათო მარკოზია",specialty:"თორაკო ქირურგია",phone:"593 100 176"},{name:"ლევან ქაცარავა",specialty:"თორაკო ქირურგია",phone:"593 696 743"},
      {name:"ირინა სვიანაძე",specialty:"პულმონოლოგია",phone:"555 539 733"},{name:"ლანა ბერია",specialty:"პულმონოლოგია",phone:"598 358 377"},
      {name:"ნინო ღრუბელაშვილი",specialty:"ზოგადი რეანიმაცია",phone:"599 943 008"},{name:"ასიკო ენუქიძე",specialty:"ზოგადი რეანიმაცია",phone:"577 101 910"},
      {name:"თიკო კუჭავა",specialty:"ზოგადი რეანიმაცია",phone:"599 425 646"},{name:"დათო კახიძე",specialty:"ზოგადი რეანიმაცია",phone:"598 535 337"},
      {name:"შორენა მურმანიშვილი",specialty:"ზოგადი რეანიმაცია",phone:"599 361 288"},{name:"თამუნა ხუციშვილი",specialty:"ზოგადი რეანიმაცია",phone:"599 141 380"},
      {name:"ლიკა ქობლიანიძე",specialty:"ზოგადი რეანიმაცია",phone:"599 313 345"},{name:"ნათია ჯიყაშვილი",specialty:"ზოგადი რეანიმაცია",phone:"592 058 180"},
      {name:"ვახტანგ ჩიქოვანი",specialty:"ზოგადი რეანიმაცია",phone:"599 420 576"},{name:"მარიამ მერებაშვილი",specialty:"ზოგადი რეანიმაცია",phone:"598 477 662"},
      {name:"დათო მამინაშვილი",specialty:"შინაგანი მედიცინა",phone:"579 49 15 51"},{name:"ნიკო გოგალაძე",specialty:"შინაგანი მედიცინა",phone:"568 96 13 85"},
      {name:"გვანცა ხაჩიაშვილი",specialty:"შინაგანი მედიცინა",phone:"571 187 920"},{name:"ნათია ეფრემიძე",specialty:"შინაგანი მედიცინა",phone:"557 752 842"},
      {name:"ნინო მიტიჩაშვილი",specialty:"ჰეპატოლოგია",phone:"579 559 558"},{name:"მარიამ ბერიძე",specialty:"ნეფროლოგია",phone:"599 758 607"},
      {name:"მაკა ტაბაღუა",specialty:"ნეფროლოგია",phone:"598 77 88 12"},{name:"მარიამ გიუაშვილი",specialty:"ნეფროლოგია",phone:"551 45 21 21"},
      {name:"რუსუდან რუსია",specialty:"ნეფროლოგია",phone:"599 11 15 11"},{name:"ნორა სარიშვილი",specialty:"ნეფროლოგია",phone:"593 128 485"},
      {name:"სალომე დარასელია",specialty:"ნეფროლოგია",phone:"574 54 37 37"},{name:"გიორგი გაზდელიანი",specialty:"ნეფროლოგია",phone:"591 000 604"},
      {name:"ანა ჭიქაბერიძე",specialty:"ნეფროლოგია",phone:"599 103 106"},{name:"ხაიალ დემურჩიევ",specialty:"ნეფროლოგია",phone:"577 591 644"},
      {name:"ნინო ბუაძე",specialty:"ნეფროლოგია",phone:"593 494 995"},{name:"თეონა ხელაშვილი",specialty:"ნეფროლოგია",phone:"557 438 626"},
      {name:"გვანცა მეცხვარიშვილი",specialty:"ნეფროლოგია",phone:"597 777 991"},{name:"თამარ კასრაძე",specialty:"ნეფროლოგია",phone:"593 329 900"},
      {name:"ნონა ბაბუციძე",specialty:"ნეფროლოგია",phone:"555 595 550"},{name:"თამარ თევდორაძე",specialty:"ნეფროლოგია",phone:"551 770 505"},
      {name:"ქეთევან დალაქიშვილი",specialty:"ნეფროლოგია",phone:"599 194 353"},{name:"თამარ ბაგაშვილი",specialty:"ნეფროლოგია",phone:"593 934 241"},
      {name:"ქეთი კაპანაძე",specialty:"ნეფროლოგია",phone:"598 232 177"},{name:"ზურაბ გოგინაშვილი",specialty:"ანგიოქირურგია",phone:"599 11 34 57"},
      {name:"ზურაბ გოგიჩაშვილი",specialty:"ანგიოქირურგია",phone:"599 55 80 60"},{name:"დათო ბაბილოძე",specialty:"ანგიოქირურგია",phone:"599 520 938"},
      {name:"ნატალია ჯინჯოლია",specialty:"კარდიოლოგია",phone:"599 158 738"},{name:"ნანა ჩადუნელი",specialty:"კარდიოლოგია",phone:"593 145 444"},
      {name:"სოფიო ნაჭყებია",specialty:"კარდიოლოგია",phone:"574 730 555"},{name:"ზურაბ ოკუჯავა",specialty:"კარდიოლოგია",phone:"599 584 646"},
      {name:"ნათია ჩიქოვანი",specialty:"კარდიოლოგია",phone:"598 280 843"},{name:"ნინო ჩხაიძე",specialty:"კარდიოლოგია",phone:"599 073 195"},
      {name:"ბაქარ ცნობილაძე",specialty:"კარდიოლოგია",phone:"568 817 537"},{name:"ნინო გიორგაძე",specialty:"კარდიოლოგია",phone:"577 970 910"},
      {name:"თინათინ ნაფეტვარიძე",specialty:"კარდიოლოგია",phone:"598 358 522"}
    ];
    doctors = [...defaultDoctors];

    // დამატებითი ექიმები Firebase-დან
    doctorsRef.on('value', s => { if(s.val()){ Object.keys(s.val()).forEach(k=>{ const d=s.val()[k]; if(!doctors.find(x=>x.name===d.name && x.phone===d.phone)) doctors.push({...d,id:k}); }); populateSpecialties(); renderDoctorList(); } });
    shiftsRef.on('value', s => { allShifts = s.val() ? Object.keys(s.val()).map(k=>({id:k,...s.val()[k]})) : []; renderCalendar(); if(selectedDate) renderShiftsForDate(selectedDate); updateStatus('მზადაა!', '#10b981'); });

    const modal=document.getElementById('shift-modal'), openBtn=document.getElementById('open-modal-btn'), closeBtn=document.getElementById('close-modal'), doctorSearch=document.getElementById('doctor-search'), specialtyFilter=document.getElementById('specialty-filter'), doctorList=document.getElementById('doctor-list'), modalPhone=document.getElementById('modal-phone'), modalDate=document.getElementById('modal-date'), modalHours=document.getElementById('modal-hours'), repeatType=document.getElementById('repeat-type'), repeatUntil=document.getElementById('repeat-until'), addFinalBtn=document.getElementById('add-shift-final'), calendarGrid=document.getElementById('calendar-grid'), monthYearEl=document.getElementById('month-year'), selectedDateTitle=document.getElementById('selected-date-title'), departmentsGrid=document.getElementById('departments-grid'), selectedDateView=document.getElementById('selected-date-view'), exportBtn=document.getElementById('export-excel'), deptSearch=document.getElementById('dept-search'), statusEl=document.getElementById('status');

    function updateStatus(m,c='#f59e0b'){ statusEl.textContent=m; statusEl.style.background=c; }
    function formatDateDDMMYYYY(d){ const [y,m,dd]=d.split('-'); return `${dd}.${m}.${y}`; }

    function populateSpecialties(){ const specs=[...new Set(doctors.map(d=>d.specialty))].sort(); specialtyFilter.innerHTML='<option value="">ყველა</option>'+specs.map(s=>`<option value="${s}">${s}</option>`).join(''); }
    function renderDoctorList(){
      const q=doctorSearch.value.toLowerCase().trim(), spec=specialtyFilter.value;
      const filtered=doctors.filter(d=>(d.name.toLowerCase().includes(q)||d.phone.includes(q))&&(!spec||d.specialty===spec));
      doctorList.innerHTML=filtered.length===0?'<div style="padding:20px;text-align:center;color:#888;">ექიმი არ მოიძებნა</div>':'';
      filtered.forEach(d=>{ const div=document.createElement('div'); div.className='doctor-item'; if(selectedDoctor&&selectedDoctor.name===d.name&&selectedDoctor.phone===d.phone) div.classList.add('selected');
        div.innerHTML=`<strong>${d.name}</strong><br><small>${d.specialty} • ${d.phone}</small>`;
        div.onclick=()=>{ selectedDoctor=d; modalPhone.value=d.phone; document.querySelectorAll('.doctor-item').forEach(i=>i.classList.remove('selected')); div.classList.add('selected'); };
        doctorList.appendChild(div);
      });
    }

    function renderCalendar(){
      const firstDay=new Date(currentYear,currentMonth,1).getDay()||7;
      const daysInMonth=new Date(currentYear,currentMonth+1,0).getDate();
      const today=new Date().toISOString().split('T')[0];
      while(calendarGrid.children.length>7) calendarGrid.removeChild(calendarGrid.lastChild);
      let day=1;
      for(let i=0;i<6;i++) for(let j=0;j<7;j++){
        if(i===0&&j<firstDay-1||day>daysInMonth) { if(day<=daysInMonth){ const empty=document.createElement('div'); empty.className='day-cell'; calendarGrid.appendChild(empty); } continue; }
        const dateStr=`${currentYear}-${String(currentMonth+1).padStart(2,'0')}-${String(day).padStart(2,'0')}`;
        const count=allShifts.filter(s=>s.date===dateStr).length;
        const cell=document.createElement('div'); cell.className='day-cell'+(dateStr===today?' today':'')+(count>0?' has-shift':'');
        cell.innerHTML=`<div class="date-num">${day}</div>${count>0?`<div class="shift-count">${count} </div>`:''}`;
        cell.onclick=()=>openDateView(dateStr);
        calendarGrid.appendChild(cell); day++;
      }
      monthYearEl.textContent=`${['იანვარი','თებერვალი','მარტი','აპრილი','მაისი','ივნისი','ივლისი','აგვისტო','სექტემბერი','ოქტომბერი','ნოემბერი','დეკემბერი'][currentMonth]} ${currentYear}`;
    }

    function openDateView(d){ selectedDate=d; modalDate.value=d; selectedDateTitle.textContent=`მორიგეები - ${formatDateDDMMYYYY(d)}`; selectedDateView.style.display='block'; renderShiftsForDate(d); }
    function renderShiftsForDate(d){
      const shifts=allShifts.filter(s=>s.date===d);
      const byDept={}; shifts.forEach(s=>{ if(!byDept[s.specialty]) byDept[s.specialty]=[]; byDept[s.specialty].push(s); });
      const search=deptSearch.value.toLowerCase(); departmentsGrid.innerHTML='';
      Object.keys(byDept).filter(x=>x.toLowerCase().includes(search)).sort().forEach(dept=>{
        const card=document.createElement('div'); card.className='dept-card'; card.innerHTML=`<div class="dept-header">${dept}</div>`;
        byDept[dept].forEach(s=>{ const item=document.createElement('div'); item.className='shift-item';
          item.innerHTML=`<div class="shift-info"><strong>${s.name}</strong><br><small>${s.phone}</small></div><div style="display:flex;gap:8px;align-items:center;"><span class="shift-hours">${s.hours} სთ</span><button class="delete-btn" onclick="shiftsRef.child('${s.id}').remove();event.stopPropagation();">X</button></div>`;
          card.appendChild(item);
        });
        departmentsGrid.appendChild(card);
      });
    }

    // სრული ახალი Excel ექსპორტი
    exportBtn.onclick=()=>{
      const wb=XLSX.utils.book_new();
      const monthPrefix=`${currentYear}-${String(currentMonth+1).padStart(2,'0')}`;
      const monthShifts=allShifts.filter(s=>s.date.startsWith(monthPrefix));
      const byDept={};
      monthShifts.forEach(s=>{ if(!byDept[s.specialty]) byDept[s.specialty]=[]; byDept[s.specialty].push(s); });

      Object.keys(byDept).sort().forEach(dept=>{
        const deptData=[];
        const doctorMap={};
        byDept[dept].forEach(s=>{
          const key=`${s.name}|||${s.phone}`;
          if(!doctorMap[key]) doctorMap[key]={name:s.name,phone:s.phone,hours:Array(32).fill('')};
          const day=parseInt(s.date.split('-')[2]);
          doctorMap[key].hours[day]=s.hours;
        });
        const daysInMonth=new Date(currentYear,currentMonth+1,0).getDate();
        const header=['ექიმი','ტელეფონი'];
        for(let d=1;d<=daysInMonth;d++){
          const date=new Date(currentYear,currentMonth,d);
          const isWeekend=date.getDay()===0||date.getDay()===6;
          header.push({v:d,t:'n',s:isWeekend?{fill:{fgColor:{rgb:'E5E7EB'}}}:{} });
        }
        header.push('მორიგეების რ-ბა','საათები');
        deptData.push(header);

        Object.values(doctorMap).forEach(doc=>{
          let count=0, total=0;
          const row=[doc.name,doc.phone];
          for(let d=1;d<=daysInMonth;d++){
            const h=doc.hours[d];
            if(h){ count++; total+=parseInt(h); }
            row.push(h||'');
          }
          row.push(count,total);
          deptData.push(row);
        });

        const ws=XLSX.utils.aoa_to_sheet(deptData);
        ws['!cols']=[{wch:25},{wch:15},...Array(daysInMonth).fill({wch:5}),{wch:14},{wch:10}];
        XLSX.utils.book_append_sheet(wb,ws,dept.length>30?dept.substring(0,30):dept);
      });

      const monthName=['იანვარი','თებერვალი','მარტი','აპრილი','მაისი','ივნისი','ივლისი','აგვისტო','სექტემბერი','ოქტომბერი','ნოემბერი','დეკემბერი'][currentMonth];
      XLSX.writeFile(wb,`მორიგეები_${monthName}_${currentYear}.xlsx`);
    };

    // მოდალი და დანარჩენი ღილაკები
    openBtn.onclick=()=>{ modal.classList.add('active'); selectedDoctor=null; modalPhone.value=''; renderDoctorList(); };
    closeBtn.onclick=()=>{ modal.classList.remove('active'); };
    document.getElementById('prev-month').onclick=()=>{ currentMonth=(currentMonth-1+12)%12; if(currentMonth===11) currentYear--; renderCalendar(); };
    document.getElementById('next-month').onclick=()=>{ currentMonth=(currentMonth+1)%12; if(currentMonth===0) currentYear++; renderCalendar(); };
    document.getElementById('today-btn').onclick=()=>{ const t=new Date(); currentMonth=t.getMonth(); currentYear=t.getFullYear(); renderCalendar(); };
    doctorSearch.oninput=renderDoctorList; specialtyFilter.onchange=renderDoctorList; deptSearch.oninput=()=>{ if(selectedDate) renderShiftsForDate(selectedDate); };
    repeatType.onchange=()=>{ repeatUntil.style.display=repeatType.value!=='none'?'block':'none'; };

    addFinalBtn.onclick=()=>{
      if(!selectedDoctor) return alert('აირჩიეთ ექიმი');
      if(!modalDate.value||!modalHours.value) return alert('შეავსეთ ყველა ველი');
      const base={name:selectedDoctor.name,specialty:selectedDoctor.specialty,phone:selectedDoctor.phone,date:modalDate.value,hours:modalHours.value};
      const days=repeatType.value==='none'?1:parseInt(repeatUntil.value)||30;
      let added=0;
      for(let i=0;i<days;i+=repeatType.value==='every2'?2:repeatType.value==='every4'?4:1){
        const d=new Date(modalDate.value); d.setDate(d.getDate()+i);
        const dateStr=d.toISOString().split('T')[0];
        shiftsRef.push({...base,date:dateStr}); added++;
      }
      modal.classList.remove('active');
      alert(`დაემატა ${added} მორიგეობა!`);
    };

    document.getElementById('add-new-doctor').onclick=()=>{
      const n=document.getElementById('new-name').value.trim(), s=document.getElementById('new-specialty-select').value, p=document.getElementById('new-phone').value.trim();
      if(!n||!s||!p) return alert('შეავსეთ ყველა ველი');
      doctorsRef.push({name:n,specialty:s,phone:p});
      document.getElementById('new-name').value=document.getElementById('new-phone').value=''; document.getElementById('new-specialty-select').value='';
      alert('ექიმი დაემატა!');
    };

    renderCalendar();
  </script>
</body>
</html>
