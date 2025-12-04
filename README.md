<!doctype html>
<html lang="th">
 <head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Health Detective Game</title>
  <script src="/_sdk/element_sdk.js"></script>
  <script src="/_sdk/data_sdk.js"></script>
  <script src="https://cdn.tailwindcss.com"></script>
  <style>
    body {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      height: 100%;
    }
    html {
      height: 100%;
    }
    .game-container {
      font-family: 'Noto Sans Thai', 'Sarabun', sans-serif;
    }
    .bounce-in {
      animation: bounceIn 0.6s cubic-bezier(0.68, -0.55, 0.265, 1.55);
    }
    @keyframes bounceIn {
      0% { opacity: 0; transform: scale(0.3); }
      50% { transform: scale(1.05); }
      100% { opacity: 1; transform: scale(1); }
    }
    .slide-in {
      animation: slideIn 0.4s ease-out;
    }
    @keyframes slideIn {
      from { opacity: 0; transform: translateX(-30px); }
      to { opacity: 1; transform: translateX(0); }
    }
    .option-card {
      transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
      cursor: pointer;
      position: relative;
      overflow: hidden;
    }
    .option-card:hover {
      transform: translateY(-4px) scale(1.02);
      box-shadow: 0 8px 20px rgba(0,0,0,0.15);
    }
    .option-card.selected {
      transform: translateY(-4px) scale(1.02);
    }
    .option-card::before {
      content: '';
      position: absolute;
      top: 0;
      left: -100%;
      width: 100%;
      height: 100%;
      background: linear-gradient(90deg, transparent, rgba(255,255,255,0.3), transparent);
      transition: left 0.5s;
    }
    .option-card:hover::before {
      left: 100%;
    }
    .progress-fill {
      transition: width 0.5s cubic-bezier(0.4, 0, 0.2, 1);
    }
    .badge {
      display: inline-block;
      animation: wiggle 1s ease-in-out infinite;
    }
    @keyframes wiggle {
      0%, 100% { transform: rotate(-3deg); }
      50% { transform: rotate(3deg); }
    }
    .score-circle {
      animation: scaleUp 0.6s cubic-bezier(0.68, -0.55, 0.265, 1.55);
    }
    @keyframes scaleUp {
      0% { transform: scale(0); }
      50% { transform: scale(1.1); }
      100% { transform: scale(1); }
    }
  </style>
  <style>@view-transition { navigation: auto; }</style>
 </head>
 <body>
  <div id="app" class="game-container w-full min-h-full"></div>
  <script>
    const defaultConfig = {
      background_color: "#fef3c7",
      surface_color: "#ffffff",
      text_color: "#292524",
      primary_action_color: "#f59e0b",
      secondary_action_color: "#84cc16",
      quiz_title: "🕵️ นักสืบสุขภาพออนไลน์",
      quiz_subtitle: "ไขปริศนาข้อมูลสุขภาพ และเป็นฮีโร่ดิจิทัล!",
      start_button_text: "🚀 เริ่มผจญภัย",
      submit_button_text: "✨ ดูผลลัพธ์",
      results_title: "🎉 ภารกิจสำเร็จ!",
      font_family: "Noto Sans Thai",
      font_size: 16
    };

    let config = { ...defaultConfig };
    let currentState = 'home';
    let currentQuestionIndex = 0;
    let answers = {};
    let studentName = '';
    let isSubmitting = false;
    let allRecords = [];

    const questions = [
      {
        id: 1,
        question: "เพื่อนส่งคลิปมาว่า 'ดื่มน้ำมะนาวร้อนตอนเช้า รักษามะเร็งได้!' คุณจะทำยังไง? 🍋",
        options: [
          { id: 'a', text: "แชร์ต่อทันที! คนอื่นน่าจะรู้", score: 0, emoji: "😱" },
          { id: 'b', text: "ลองดื่มดู แล้วค่อยว่ากัน", score: 1, emoji: "🤔" },
          { id: 'c', text: "ค้นหาข้อมูลจากเว็บไซต์โรงพยาบาลหรือหมอก่อน", score: 3, emoji: "🔍" },
          { id: 'd', text: "ไม่สนใจ ข้ามไป", score: 2, emoji: "😐" }
        ]
      },
      {
        id: 2,
        question: "เห็นโฆษณายาลดน้ำหนัก 10 กิโลใน 7 วัน ในโซเชียล คุณจะ... 💊",
        options: [
          { id: 'a', text: "สนใจมาก! กดสั่งเลย", score: 0, emoji: "💸" },
          { id: 'b', text: "อ่านรีวิวในคอมเมนต์ดูก่อน", score: 1, emoji: "📱" },
          { id: 'c', text: "สงสัยทันที ดูมั่วๆ", score: 3, emoji: "🚨" },
          { id: 'd', text: "ไปปรึกษา���มอหรือเภสัชกรก่อน", score: 3, emoji: "👨‍⚕️" }
        ]
      },
      {
        id: 3,
        question: "คุณปวดหัวมาก 2 วัน จะหาข้อมูลจากที่ไหน? 🤕",
        options: [
          { id: 'a', text: "Google แล้วอ่านบทความแรกที่เจอ", score: 1, emoji: "🔍" },
          { id: 'b', text: "ถามในกลุ่มเฟสบุ๊ก", score: 1, emoji: "💬" },
          { id: 'c', text: "เว็บไซต์โรงพยาบาล หรือแอปหมอชนะ", score: 3, emoji: "🏥" },
          { id: 'd', text: "ไม่ค้นหาหรอก ทนไปก่อน", score: 0, emoji: "😣" }
        ]
      },
      {
        id: 4,
        question: "เพื่อนบอกว่า 'วัคซีนโควิดมีชิปติดตั้ง' คุณจะตอบว่า... 💉",
        options: [
          { id: 'a', text: "จริงหรอ?! ไม่ฉีดดีกว่า", score: 0, emoji: "😰" },
          { id: 'b', text: "ขอดูหลักฐาน��ากแหล่งที่เชื่อถือได้สิ", score: 3, emoji: "🎯" },
          { id: 'c', text: "ไม่แน่ใจ แต่กลัวๆ", score: 1, emoji: "😟" },
          { id: 'd', text: "นี่มันข่าวปลอมชัดๆ!", score: 3, emoji: "✋" }
        ]
      },
      {
        id: 5,
        question: "แอปสุขภาพตัวไหนที่คุณไว้ใจได้? 📱",
        options: [
          { id: 'a', text: "แอปยอดนิยมที่มีคนดาวน์โหลดเยอะๆ", score: 1, emoji: "⭐" },
          { id: 'b', text: "แอปจากโรงพยาบาลหรือหน่วยงานราชการ", score: 3, emoji: "✅" },
          { id: 'c', text: "ไม่เคยใช้แอปสุขภาพเลย", score: 0, emoji: "��" },
          { id: 'd', text: "แอปที่เพื่อน���นะนำมา", score: 1, emoji: "👥" }
        ]
      },
      {
        id: 6,
        question: "เห็นเว็บไซต์ขายยา ถูกกว่าร้านยามาก คุณจะ... 💳",
        options: [
          { id: 'a', text: "ซื้อเลย! ประหยัดได้เยอะ", score: 0, emoji: "🛒" },
          { id: 'b', text: "ดูว่ามีใบอนุญาตขายยาออนไลน์ไหม", score: 3, emoji: "📜" },
          { id: 'c', text: "ไม่กล้าซื้อออนไลน์", score: 1, emoji: "😬" },
          { id: 'd', text: "ถูกแบบนี้ น่าสงสัย ไม่ซื้อดีกว่า", score: 2, emoji: "🤨" }
        ]
      },
      {
        id: 7,
        question: "อินฟลูเอนเซอร์รีวิว���ินค้าสุขภาพ 'ของดีต้องบอกต่���!' คุณคิดว่า... 🌟",
        options: [
          { id: 'a', text: "เชื่อเลย คนดังใช้แล้วดีแน่ๆ", score: 0, emoji: "🤩" },
          { id: 'b', text: "เป็นแค่โฆษณา ต้องหาข้อมูลเพิ่ม", score: 3, emoji: "🧐" },
          { id: 'c', text: "ลองดูว่ามีการรับรองจาก��มอไหม", score: 3, emoji: "👨‍⚕️" },
          { id: 'd', text: "ไม่สนใจพวกนี้หรอก", score: 2, emoji: "😑" }
        ]
      },
      {
        id: 8,
        question: "เจอข้อมูลสุขภาพ 2 แหล่งขัดแย้งกัน คุณจะ... 🤔",
        options: [
          { id: 'a', text: "เชื่อตัวที่มีคนอ่านเยอะกว่า", score: 1, emoji: "👀" },
          { id: 'b', text: "หาข้อมูลจากหลายแหล่ง เทียบกันดู", score: 3, emoji: "📚" },
          { id: 'c', text: "สับสน! ไม่เอาละ", score: 0, emoji: "😵" },
          { id: 'd', text: "ถามหมอหรือเภสัชกรจริงๆ ดีกว่า", score: 3, emoji: "💡" }
        ]
      }
    ];

    const dataHandler = {
      onDataChanged(data) {
        allRecords = data || [];
        if (currentState === 'dashboard') {
          render();
        }
      }
    };

    function calculateResults(answers) {
      let totalScore = 0;
      let maxScore = 0;

      questions.forEach(q => {
        maxScore += 3;
        if (answers[q.id]) {
          const selected = q.options.find(opt => opt.id === answers[q.id]);
          if (selected) totalScore += selected.score;
        }
      });

      const percentage = (totalScore / maxScore) * 100;
      let level, badge, recommendation;

      if (percentage >= 80) {
        level = "ยอดนักสืบ";
        badge = "🏆";
        recommendation = "เยี่ยมเลย! คุณคือนักสืบข้อมูลสุขภาพตัวจริง รู้จักแยกแยะข้อมูลจริง-เท็จ และใช้แหล่งข้อม���ล��ี่เชื่อถือได้ ช่วยเพื่อนๆ รอบตัวด้วยนะ!";
      } else if (percentage >= 60) {
        level = "นักสืบมือดี";
        badge = "🎖️";
        recommendation = "เก่งมาก! คุณมีพื้นฐานที่ดี ลองฝึกเช็คข้อมูลจากหลายแหล่งเพิ่มเติม และถามหมอเมื่อไม่แน่ใจ เร็วๆ นี้คุณจะเป็นยอดนักสืบแน่นอน!";
      } else if (percentage >= 40) {
        level = "นักสืบฝึกหัด";
        badge = "🔰";
        recommendation = "เริ่มต้นได้ดีแล้ว! ลองฝึกตั้งคำถามกับข้อมูลที่เจอ อย่าเพิ่งเชื่อง่ายๆ และเรียนรู้หาแห���่งข้อมูลที่เชื่อถือได้นะ จะเก่งขึ้นแน���นอน!";
      } else {
        level = "นักสืบมือใหม่";
        badge = "🌱";
        recommendation = "ไม่เป็นไร! ทุกคนเริ่มต้นที่จุดนี้ ลองเล่นอีกครั้งแล้วสังเกตคำใบ้ เมื่อเจอข้อมูลสุขภาพ อย่าลืมถามหมอหรือใช้เว็บไซต์ที่เชื่อถือได้นะ!";
      }

      return { totalScore, maxScore, percentage, level, badge, recommendation };
    }

    function renderHome() {
      const baseSize = config.font_size || defaultConfig.font_size;
      const fontFamily = config.font_family || defaultConfig.font_family;
      const backgroundColor = config.background_color || defaultConfig.background_color;
      const surfaceColor = config.surface_color || defaultConfig.surface_color;
      const textColor = config.text_color || defaultConfig.text_color;
      const primaryColor = config.primary_action_color || defaultConfig.primary_action_color;
      const secondaryColor = config.secondary_action_color || defaultConfig.secondary_action_color;
      const quizTitle = config.quiz_title || defaultConfig.quiz_title;
      const quizSubtitle = config.quiz_subtitle || defaultConfig.quiz_subtitle;
      const startButtonText = config.start_button_text || defaultConfig.start_button_text;

      return `
        <div class="w-full min-h-full flex items-center justify-center p-6" style="background: linear-gradient(135deg, ${backgroundColor} 0%, ${surfaceColor} 100%); font-family: '${fontFamily}', sans-serif;">
          <div class="w-full max-w-2xl bounce-in">
            <div class="rounded-3xl shadow-2xl p-8 border-4" style="background-color: ${surfaceColor}; border-color: ${primaryColor};">
              <div class="text-center mb-8">
                <div class="badge text-6xl mb-4">🕵️‍♂️</div>
                <h1 class="font-bold mb-3" style="color: ${textColor}; font-size: ${baseSize * 2}px;">${quizTitle}</h1>
                <p class="mb-6" style="color: ${textColor}; font-size: ${baseSize * 1.1}px; opacity: 0.8;">${quizSubtitle}</p>
              </div>

              <div class="mb-6 p-6 rounded-2xl" style="background: linear-gradient(135deg, ${primaryColor}15 0%, ${secondaryColor}15 100%);">
                <div class="flex items-center gap-3 mb-4">
                  <span style="font-size: ${baseSize * 1.5}px;">🎯</span>
                  <span style="color: ${textColor}; font-size: ${baseSize}px; font-weight: 600;">ภารกิจของคุณ:</span>
                </div>
                <ul class="space-y-2" style="color: ${textColor}; font-size: ${baseSize * 0.95}px; padding-left: 20px;">
                  <li>✓ ตอบสถานการณ์ 8 ข้อ</li>
                  <li>✓ แยกแยะข้อมูลจริง-เท็จ</li>
                  <li>✓ เป็นฮีโร่สุขภาพดิจิทัล!</li>
                </ul>
              </div>

              <form id="nameForm" class="mb-4">
                <label for="studentName" class="block mb-2 font-semibold" style="color: ${textColor}; font-size: ${baseSize * 1.05}px;">👤 ชื่อของคุณ</label>
                <input 
                  type="text" 
                  id="studentName" 
                  required
                  class="w-full px-4 py-3 rounded-xl border-3 transition-all focus:outline-none"
                  style="border: 3px solid ${primaryColor}; color: ${textColor}; font-size: ${baseSize}px;"
                  placeholder="กรอกชื่อเล่นของคุณ"
                >
                <button 
                  type="submit"
                  class="w-full mt-4 px-6 py-4 rounded-xl font-bold transition-all transform hover:scale-105 shadow-lg"
                  style="background: linear-gradient(135deg, ${primaryColor} 0%, ${secondaryColor} 100%); color: white; font-size: ${baseSize * 1.2}px;"
                >${startButtonText}</button>
              </form>

              <button 
                id="dashboardBtn"
                class="w-full px-4 py-3 rounded-xl font-semibold transition-all transform hover:scale-105 border-2"
                style="background-color: ${surfaceColor}; border-color: ${secondaryColor}; color: ${secondaryColor}; font-size: ${baseSize}px;"
              >📊 แดชบอร์ดครู/ผู้บริหาร</button>

              <div class="grid grid-cols-3 gap-4 text-center pt-6 border-t-2" style="border-color: ${textColor}; opacity: 0.2;">
                <div style="color: ${textColor};">
                  <div class="text-3xl mb-1">⏱️</div>
                  <div style="font-size: ${baseSize * 0.85}px; opacity: 0.7;">5 นาที</div>
                </div>
                <div style="color: ${textColor};">
                  <div class="text-3xl mb-1">❓</div>
                  <div style="font-size: ${baseSize * 0.85}px; opacity: 0.7;">8 คำถาม</div>
                </div>
                <div style="color: ${textColor};">
                  <div class="text-3xl mb-1">🎁</div>
                  <div style="font-size: ${baseSize * 0.85}px; opacity: 0.7;">รับตราสัญลักษณ์</div>
                </div>
              </div>
            </div>
          </div>
        </div>
      `;
    }

    function renderQuiz() {
      const question = questions[currentQuestionIndex];
      const progress = ((currentQuestionIndex + 1) / questions.length) * 100;
      const baseSize = config.font_size || defaultConfig.font_size;
      const fontFamily = config.font_family || defaultConfig.font_family;
      const backgroundColor = config.background_color || defaultConfig.background_color;
      const surfaceColor = config.surface_color || defaultConfig.surface_color;
      const textColor = config.text_color || defaultConfig.text_color;
      const primaryColor = config.primary_action_color || defaultConfig.primary_action_color;
      const secondaryColor = config.secondary_action_color || defaultConfig.secondary_action_color;
      const submitButtonText = config.submit_button_text || defaultConfig.submit_button_text;

      const optionsHtml = question.options.map((opt, index) => {
        const isSelected = answers[question.id] === opt.id;
        return `
          <div 
            class="option-card slide-in rounded-2xl p-5 border-3 ${isSelected ? 'selected' : ''}"
            data-question="${question.id}" 
            data-option="${opt.id}"
            style="
              background-color: ${isSelected ? primaryColor + '20' : surfaceColor}; 
              border: 3px solid ${isSelected ? primaryColor : textColor + '30'}; 
              color: ${textColor}; 
              font-size: ${baseSize}px;
              animation-delay: ${index * 0.1}s;
              box-shadow: ${isSelected ? '0 8px 20px rgba(0,0,0,0.2)' : '0 2px 8px rgba(0,0,0,0.1)'};
            "
          >
            <div class="flex items-center gap-3">
              <div class="text-3xl flex-shrink-0">${opt.emoji}</div>
              <span style="line-height: 1.5;">${opt.text}</span>
              ${isSelected ? `<div class="ml-auto text-2xl">✓</div>` : ''}
            </div>
          </div>
        `;
      }).join('');

      const isLastQuestion = currentQuestionIndex === questions.length - 1;
      const buttonText = isLastQuestion ? submitButtonText : '➡️ คำถามถัดไป';

      return `
        <div class="w-full min-h-full p-6" style="background: linear-gradient(135deg, ${backgroundColor} 0%, ${surfaceColor} 100%); font-family: '${fontFamily}', sans-serif;">
          <div class="w-full max-w-3xl mx-auto">
            <div class="mb-6 bounce-in">
              <div class="flex justify-between items-center mb-3" style="color: ${textColor}; font-size: ${baseSize}px; font-weight: 600;">
                <span>🎯 ด่าน ${currentQuestionIndex + 1}/${questions.length}</span>
                <span>${Math.round(progress)}%</span>
              </div>
              <div class="w-full h-4 rounded-full overflow-hidden" style="background-color: ${surfaceColor}; box-shadow: inset 0 2px 4px rgba(0,0,0,0.1);">
                <div class="progress-fill h-full rounded-full" style="width: ${progress}%; background: linear-gradient(90deg, ${primaryColor} 0%, ${secondaryColor} 100%);"></div>
              </div>
            </div>

            <div class="rounded-3xl shadow-2xl p-8 mb-6 border-4 bounce-in" style="background-color: ${surfaceColor}; border-color: ${primaryColor};">
              <div class="flex items-start gap-4 mb-6">
                <div class="text-4xl flex-shrink-0">🤔</div>
                <h2 class="font-bold leading-relaxed" style="color: ${textColor}; font-size: ${baseSize * 1.4}px;">${question.question}</h2>
              </div>
              
              <div class="space-y-3">
                ${optionsHtml}
              </div>
            </div>

            <div class="flex gap-4">
              ${currentQuestionIndex > 0 ? `
                <button 
                  id="prevBtn"
                  class="px-6 py-3 rounded-xl font-semibold transition-all transform hover:scale-105 shadow-lg"
                  style="background-color: ${secondaryColor}; color: white; font-size: ${baseSize}px;"
                >⬅️ ย้อนกลับ</button>
              ` : ''}
              <button 
                id="nextBtn"
                class="flex-1 px-6 py-4 rounded-xl font-bold transition-all transform hover:scale-105 shadow-lg disabled:opacity-50 disabled:cursor-not-allowed"
                style="background: linear-gradient(135deg, ${primaryColor} 0%, ${secondaryColor} 100%); color: white; font-size: ${baseSize * 1.1}px;"
                ${!answers[question.id] ? 'disabled' : ''}
              >${buttonText}</button>
            </div>
          </div>
        </div>
      `;
    }

    function renderDashboard() {
      const baseSize = config.font_size || defaultConfig.font_size;
      const fontFamily = config.font_family || defaultConfig.font_family;
      const backgroundColor = config.background_color || defaultConfig.background_color;
      const surfaceColor = config.surface_color || defaultConfig.surface_color;
      const textColor = config.text_color || defaultConfig.text_color;
      const primaryColor = config.primary_action_color || defaultConfig.primary_action_color;
      const secondaryColor = config.secondary_action_color || defaultConfig.secondary_action_color;

      // Calculate statistics
      const totalStudents = allRecords.length;
      const averageScore = totalStudents > 0 
        ? (allRecords.reduce((sum, r) => sum + r.score, 0) / totalStudents).toFixed(1)
        : 0;
      
      const levelCounts = {
        'ยอดนักสืบ': 0,
        'นักสืบมือดี': 0,
        'นักสืบฝึกหัด': 0,
        'นักสืบมือใหม่': 0
      };
      
      allRecords.forEach(record => {
        if (levelCounts[record.level] !== undefined) {
          levelCounts[record.level]++;
        }
      });

      const maxScore = 24;
      const averagePercentage = totalStudents > 0 
        ? ((allRecords.reduce((sum, r) => sum + r.score, 0) / totalStudents) / maxScore * 100).toFixed(1)
        : 0;

      // Sort records by score descending
      const sortedRecords = [...allRecords].sort((a, b) => b.score - a.score);
      const recentRecords = sortedRecords.slice(0, 10);

      const recordsHtml = recentRecords.map((record, index) => {
        const percentage = (record.score / maxScore * 100).toFixed(0);
        const date = new Date(record.completed_at);
        const dateStr = date.toLocaleDateString('th-TH', { day: 'numeric', month: 'short', year: 'numeric' });
        const timeStr = date.toLocaleTimeString('th-TH', { hour: '2-digit', minute: '2-digit' });
        
        let levelColor = primaryColor;
        let levelEmoji = '🏆';
        if (record.level === 'นักสืบมือดี') {
          levelColor = secondaryColor;
          levelEmoji = '🎖️';
        } else if (record.level === 'นักสืบฝึกหัด') {
          levelColor = '#6b7280';
          levelEmoji = '🔰';
        } else if (record.level === 'นักสืบมือใหม่') {
          levelColor = '#9ca3af';
          levelEmoji = '🌱';
        }

        return `
          <div class="flex items-center gap-4 p-4 rounded-xl border-2 slide-in" style="background-color: ${surfaceColor}; border-color: ${textColor}20; animation-delay: ${index * 0.05}s;">
            <div class="text-2xl font-bold" style="color: ${textColor}; opacity: 0.3; min-width: 30px;">#${index + 1}</div>
            <div class="flex-1">
              <div class="font-semibold mb-1" style="color: ${textColor}; font-size: ${baseSize * 1.05}px;">${record.student_name}</div>
              <div class="flex items-center gap-2 flex-wrap" style="font-size: ${baseSize * 0.85}px;">
                <span style="color: ${levelColor}; font-weight: 600;">${levelEmoji} ${record.level}</span>
                <span style="color: ${textColor}; opacity: 0.5;">•</span>
                <span style="color: ${textColor}; opacity: 0.7;">${dateStr} ${timeStr}</span>
              </div>
            </div>
            <div class="text-center">
              <div class="font-bold mb-1" style="color: ${primaryColor}; font-size: ${baseSize * 1.5}px;">${percentage}%</div>
              <div style="color: ${textColor}; opacity: 0.6; font-size: ${baseSize * 0.8}px;">${record.score}/${maxScore}</div>
            </div>
          </div>
        `;
      }).join('');

      return `
        <div class="w-full min-h-full p-6" style="background: linear-gradient(135deg, ${backgroundColor} 0%, ${surfaceColor} 100%); font-family: '${fontFamily}', sans-serif;">
          <div class="w-full max-w-6xl mx-auto">
            <div class="mb-6 flex items-center justify-between bounce-in">
              <div>
                <h1 class="font-bold mb-2" style="color: ${textColor}; font-size: ${baseSize * 2}px;">📊 แดชบอร์ดครู/ผู้บริหาร</h1>
                <p style="color: ${textColor}; font-size: ${baseSize}px; opacity: 0.7;">ภาพรวมความรอบรู้ทางสุขภาพอิเล็กทรอนิกส์ของนักเรียน</p>
              </div>
              <button 
                id="backHomeBtn"
                class="px-6 py-3 rounded-xl font-semibold transition-all transform hover:scale-105 shadow-lg"
                style="background-color: ${secondaryColor}; color: white; font-size: ${baseSize}px;"
              >← กลับหน้าแรก</button>
            </div>

            ${totalStudents === 0 ? `
              <div class="text-center p-12 rounded-2xl bounce-in" style="background-color: ${surfaceColor}; border: 2px dashed ${textColor}30;">
                <div class="text-6xl mb-4">📭</div>
                <h3 class="font-bold mb-2" style="color: ${textColor}; font-size: ${baseSize * 1.3}px;">ยังไม่มีข้อมูลนักเรียน</h3>
                <p style="color: ${textColor}; opacity: 0.7; font-size: ${baseSize}px;">เมื่อนักเรียนเล่นเกมและส่งคำตอบแล้ว ข้อมูลจะแสดงที่นี่</p>
              </div>
            ` : `
              <!-- Summary Cards -->
              <div class="grid grid-cols-1 md:grid-cols-4 gap-4 mb-6">
                <div class="p-6 rounded-2xl shadow-lg bounce-in" style="background: linear-gradient(135deg, ${primaryColor} 0%, ${secondaryColor} 100%); color: white;">
                  <div class="text-3xl mb-2">👥</div>
                  <div class="font-bold mb-1" style="font-size: ${baseSize * 2}px;">${totalStudents}</div>
                  <div style="font-size: ${baseSize * 0.9}px; opacity: 0.9;">นักเรียนทั้งหมด</div>
                </div>

                <div class="p-6 rounded-2xl shadow-lg bounce-in" style="background-color: ${surfaceColor}; animation-delay: 0.1s;">
                  <div class="text-3xl mb-2">📈</div>
                  <div class="font-bold mb-1" style="color: ${primaryColor}; font-size: ${baseSize * 2}px;">${averagePercentage}%</div>
                  <div style="color: ${textColor}; font-size: ${baseSize * 0.9}px; opacity: 0.7;">คะแนนเฉลี่ย</div>
                </div>

                <div class="p-6 rounded-2xl shadow-lg bounce-in" style="background-color: ${surfaceColor}; animation-delay: 0.2s;">
                  <div class="text-3xl mb-2">🏆</div>
                  <div class="font-bold mb-1" style="color: ${secondaryColor}; font-size: ${baseSize * 2}px;">${levelCounts['ยอดนักสืบ']}</div>
                  <div style="color: ${textColor}; font-size: ${baseSize * 0.9}px; opacity: 0.7;">ยอดนักสืบ</div>
                </div>

                <div class="p-6 rounded-2xl shadow-lg bounce-in" style="background-color: ${surfaceColor}; animation-delay: 0.3s;">
                  <div class="text-3xl mb-2">🎖️</div>
                  <div class="font-bold mb-1" style="color: ${primaryColor}; font-size: ${baseSize * 2}px;">${levelCounts['นักสืบมือดี']}</div>
                  <div style="color: ${textColor}; font-size: ${baseSize * 0.9}px; opacity: 0.7;">นักสืบมือดี</div>
                </div>
              </div>

              <!-- Distribution Chart -->
              <div class="p-6 rounded-2xl shadow-lg mb-6 bounce-in" style="background-color: ${surfaceColor}; animation-delay: 0.4s;">
                <h3 class="font-bold mb-4 flex items-center gap-2" style="color: ${textColor}; font-size: ${baseSize * 1.3}px;">
                  <span>📊</span> การกระจายระดับความสามารถ
                </h3>
                <div class="space-y-3">
                  ${Object.entries(levelCounts).map(([level, count]) => {
                    const percentage = totalStudents > 0 ? (count / totalStudents * 100).toFixed(1) : 0;
                    let barColor = primaryColor;
                    let emoji = '🏆';
                    if (level === 'นักสืบมือดี') { barColor = secondaryColor; emoji = '🎖️'; }
                    else if (level === 'นักสืบฝึกหัด') { barColor = '#6b7280'; emoji = '🔰'; }
                    else if (level === 'นักสืบมือใหม่') { barColor = '#9ca3af'; emoji = '🌱'; }
                    
                    return `
                      <div>
                        <div class="flex justify-between items-center mb-2" style="font-size: ${baseSize * 0.95}px;">
                          <span style="color: ${textColor}; font-weight: 600;">${emoji} ${level}</span>
                          <span style="color: ${textColor}; opacity: 0.7;">${count} คน (${percentage}%)</span>
                        </div>
                        <div class="w-full h-3 rounded-full" style="background-color: ${textColor}15;">
                          <div class="h-full rounded-full transition-all" style="width: ${percentage}%; background-color: ${barColor};"></div>
                        </div>
                      </div>
                    `;
                  }).join('')}
                </div>
              </div>

              <!-- Recommendations -->
              <div class="p-6 rounded-2xl shadow-lg mb-6 bounce-in" style="background: linear-gradient(135deg, ${primaryColor}15 0%, ${secondaryColor}15 100%); animation-delay: 0.5s;">
                <h3 class="font-bold mb-4 flex items-center gap-2" style="color: ${textColor}; font-size: ${baseSize * 1.3}px;">
                  <span>💡</span> คำแนะนำสำหรับครู
                </h3>
                <div class="space-y-3" style="color: ${textColor}; font-size: ${baseSize * 0.95}px; line-height: 1.7;">
                  ${averagePercentage < 60 ? `
                    <div class="flex items-start gap-2">
                      <span class="flex-shrink-0">⚠️</span>
                      <span><strong>ควรให้ความรู้เพิ่มเติม:</strong> คะแนนเฉลี่ยต่ำกว่า 60% แนะนำให้จัดกิจกรรมสอนการแยกแยะข้อมูลสุขภาพที่เชื่อถือได้</span>
                    </div>
                  ` : ''}
                  ${levelCounts['นักสืบมือใหม่'] > totalStudents * 0.2 ? `
                    <div class="flex items-start gap-2">
                      <span class="flex-shrink-0">📚</span>
                      <span><strong>กลุ่มที่ต้องการความช่วยเหลือ:</strong> มีนักเรียน ${levelCounts['นักสืบมือใหม่']} คน ที่ต้องการพัฒนาทักษะพื้นฐาน ควรให้คำแนะนำเป็นพิเศษ</span>
                    </div>
                  ` : ''}
                  <div class="flex items-start gap-2">
                    <span class="flex-shrink-0">✅</span>
                    <span><strong>แนะนำแหล่งเรียนรู้:</strong> กระทรวงสาธารณสุข (moph.go.th), แอปหมอชนะ, ศูนย์ข้อมูลสุขภาพกรมอนามัย</span>
                  </div>
                  <div class="flex items-start gap-2">
                    <span class="flex-shrink-0">🎯</span>
                    <span><strong>กิจกรรมแนะนำ:</strong> จัดเวิร์กช็อปการตรวจสอบข่าวปลอม, ฝึกหาแหล่งข้อมูลที่เชื่อถือได้, เชิญผู้เชี่ยวชาญมาให้ความรู้</span>
                  </div>
                </div>
              </div>

              <!-- Student Records -->
              <div class="p-6 rounded-2xl shadow-lg bounce-in" style="background-color: ${surfaceColor}; animation-delay: 0.6s;">
                <h3 class="font-bold mb-4 flex items-center gap-2" style="color: ${textColor}; font-size: ${baseSize * 1.3}px;">
                  <span>📋</span> รายชื่อนักเรียน ${recentRecords.length > 0 ? `(แสดง ${recentRecords.length} คนล่าสุด)` : ''}
                </h3>
                <div class="space-y-3">
                  ${recordsHtml}
                </div>
              </div>
            `}
          </div>
        </div>
      `;
    }

    function renderResults() {
      const results = calculateResults(answers);
      const baseSize = config.font_size || defaultConfig.font_size;
      const fontFamily = config.font_family || defaultConfig.font_family;
      const backgroundColor = config.background_color || defaultConfig.background_color;
      const surfaceColor = config.surface_color || defaultConfig.surface_color;
      const textColor = config.text_color || defaultConfig.text_color;
      const primaryColor = config.primary_action_color || defaultConfig.primary_action_color;
      const secondaryColor = config.secondary_action_color || defaultConfig.secondary_action_color;
      const resultsTitle = config.results_title || defaultConfig.results_title;

      return `
        <div class="w-full min-h-full flex items-center justify-center p-6" style="background: linear-gradient(135deg, ${backgroundColor} 0%, ${surfaceColor} 100%); font-family: '${fontFamily}', sans-serif;">
          <div class="w-full max-w-2xl bounce-in">
            <div class="rounded-3xl shadow-2xl p-8 border-4" style="background-color: ${surfaceColor}; border-color: ${primaryColor};">
              <div class="text-center mb-8">
                <div class="badge text-7xl mb-4">${results.badge}</div>
                <h1 class="font-bold mb-2" style="color: ${textColor}; font-size: ${baseSize * 2}px;">${resultsTitle}</h1>
                <p class="mb-2" style="color: ${textColor}; font-size: ${baseSize * 1.3}px; opacity: 0.8;">คุณ ${studentName}</p>
              </div>

              <div class="mb-8 p-8 rounded-2xl relative overflow-hidden" style="background: linear-gradient(135deg, ${primaryColor} 0%, ${secondaryColor} 100%); color: white;">
                <div class="text-center relative z-10">
                  <div class="score-circle inline-block">
                    <div class="font-bold mb-2" style="font-size: ${baseSize * 4}px;">${Math.round(results.percentage)}%</div>
                  </div>
                  <div class="mb-3" style="font-size: ${baseSize * 1.1}px; opacity: 0.95;">คะแนน ${results.totalScore}/${results.maxScore}</div>
                  <div class="inline-block px-6 py-3 rounded-full font-bold text-center" style="background-color: rgba(255,255,255,0.3); font-size: ${baseSize * 1.3}px; backdrop-filter: blur(10px);">
                    ระดับ: ${results.level}
                  </div>
                </div>
              </div>

              <div class="mb-8 p-6 rounded-2xl border-3" style="border-color: ${primaryColor}; background-color: ${primaryColor}15;">
                <div class="flex items-center gap-2 mb-4">
                  <span style="font-size: ${baseSize * 1.8}px;">💬</span>
                  <h3 class="font-bold" style="color: ${textColor}; font-size: ${baseSize * 1.3}px;">ข้อความจากหมอ</h3>
                </div>
                <p style="color: ${textColor}; font-size: ${baseSize}px; line-height: 1.8;">${results.recommendation}</p>
              </div>

              <div class="mb-8 p-6 rounded-2xl" style="background: linear-gradient(135deg, ${secondaryColor}20 0%, ${primaryColor}20 100%);">
                <h3 class="font-bold mb-4 flex items-center gap-2" style="color: ${textColor}; font-size: ${baseSize * 1.2}px;">
                  <span>🎯</span> เคล็ดลับนักสืบมืออาชีพ
                </h3>
                <div class="space-y-3" style="color: ${textColor}; font-size: ${baseSize * 0.95}px;">
                  <div class="flex items-start gap-2">
                    <span class="flex-shrink-0">✅</span>
                    <span>เช็คข้อมูลจากเว็บโรงพยาบาลหรือกระทรว��สาธารณสุข</span>
                  </div>
                  <div class="flex items-start gap-2">
                    <span class="flex-shrink-0">✅</span>
                    <span>ระวังข้อมูลที่อ้างว่า "รักษาได้ 100%" หรือ "ผลทันที"</span>
                  </div>
                  <div class="flex items-start gap-2">
                    <span class="flex-shrink-0">✅</span>
                    <span>เทียบข้อมูลจากหลายแหล่งก่อนเชื่อ</span>
                  </div>
                  <div class="flex items-start gap-2">
                    <span class="flex-shrink-0">✅</span>
                    <span>ถามหมอหรือเภสัช���รเมื่อไม่แน่ใจ</span>
                  </div>
                </div>
              </div>

              <button 
                id="restartBtn"
                class="w-full px-6 py-4 rounded-xl font-bold transition-all transform hover:scale-105 shadow-lg"
                style="background: linear-gradient(135deg, ${primaryColor} 0%, ${secondaryColor} 100%); color: white; font-size: ${baseSize * 1.2}px;"
              >🔄 เล่นอีกครั้ง</button>
            </div>
          </div>
        </div>
      `;
    }

    async function render() {
      const app = document.getElementById('app');
      if (!app) return;

      if (currentState === 'dashboard') {
        app.innerHTML = renderDashboard();
        const backHomeBtn = document.getElementById('backHomeBtn');
        if (backHomeBtn) {
          backHomeBtn.addEventListener('click', () => {
            currentState = 'home';
            render();
          });
        }
      } else if (currentState === 'home') {
        app.innerHTML = renderHome();
        const form = document.getElementById('nameForm');
        if (form) {
          form.addEventListener('submit', async (e) => {
            e.preventDefault();
            const input = document.getElementById('studentName');
            if (input && input.value.trim()) {
              studentName = input.value.trim();
              currentState = 'quiz';
              await render();
            }
          });
        }
        
        const dashboardBtn = document.getElementById('dashboardBtn');
        if (dashboardBtn) {
          dashboardBtn.addEventListener('click', () => {
            currentState = 'dashboard';
            render();
          });
        }
      } else if (currentState === 'quiz') {
        app.innerHTML = renderQuiz();
        
        const options = document.querySelectorAll('.option-card');
        options.forEach(opt => {
          opt.addEventListener('click', () => {
            const questionId = parseInt(opt.dataset.question);
            const optionId = opt.dataset.option;
            answers[questionId] = optionId;
            render();
          });
        });

        const nextBtn = document.getElementById('nextBtn');
        if (nextBtn) {
          nextBtn.addEventListener('click', async () => {
            if (isSubmitting) return;
            
            if (currentQuestionIndex === questions.length - 1) {
              isSubmitting = true;
              nextBtn.disabled = true;
              nextBtn.textContent = '⏳ กำลังคำนวณ...';
              
              const results = calculateResults(answers);
              
              if (window.dataSdk) {
                const result = await window.dataSdk.create({
                  student_name: studentName,
                  score: results.totalScore,
                  level: results.level,
                  completed_at: new Date().toISOString()
                });
                
                if (!result.isOk) {
                  console.error('Failed to save results');
                }
              }
              
              currentState = 'results';
              isSubmitting = false;
              await render();
            } else {
              currentQuestionIndex++;
              await render();
            }
          });
        }

        const prevBtn = document.getElementById('prevBtn');
        if (prevBtn) {
          prevBtn.addEventListener('click', () => {
            currentQuestionIndex--;
            render();
          });
        }
      } else if (currentState === 'results') {
        app.innerHTML = renderResults();
        const restartBtn = document.getElementById('restartBtn');
        if (restartBtn) {
          restartBtn.addEventListener('click', () => {
            currentState = 'home';
            currentQuestionIndex = 0;
            answers = {};
            studentName = '';
            render();
          });
        }
      }
    }

    async function onConfigChange(newConfig) {
      Object.assign(config, newConfig);
      await render();
    }

    if (window.dataSdk) {
      window.dataSdk.init(dataHandler).then(result => {
        if (!result.isOk) {
          console.error('Failed to initialize data SDK');
        }
      });
    }

    if (window.elementSdk) {
      window.elementSdk.init({
        defaultConfig,
        onConfigChange,
        mapToCapabilities: (config) => ({
          recolorables: [
            {
              get: () => config.background_color || defaultConfig.background_color,
              set: (value) => {
                config.background_color = value;
                window.elementSdk.setConfig({ background_color: value });
              }
            },
            {
              get: () => config.surface_color || defaultConfig.surface_color,
              set: (value) => {
                config.surface_color = value;
                window.elementSdk.setConfig({ surface_color: value });
              }
            },
            {
              get: () => config.text_color || defaultConfig.text_color,
              set: (value) => {
                config.text_color = value;
                window.elementSdk.setConfig({ text_color: value });
              }
            },
            {
              get: () => config.primary_action_color || defaultConfig.primary_action_color,
              set: (value) => {
                config.primary_action_color = value;
                window.elementSdk.setConfig({ primary_action_color: value });
              }
            },
            {
              get: () => config.secondary_action_color || defaultConfig.secondary_action_color,
              set: (value) => {
                config.secondary_action_color = value;
                window.elementSdk.setConfig({ secondary_action_color: value });
              }
            }
          ],
          borderables: [],
          fontEditable: {
            get: () => config.font_family || defaultConfig.font_family,
            set: (value) => {
              config.font_family = value;
              window.elementSdk.setConfig({ font_family: value });
            }
          },
          fontSizeable: {
            get: () => config.font_size || defaultConfig.font_size,
            set: (value) => {
              config.font_size = value;
              window.elementSdk.setConfig({ font_size: value });
            }
          }
        }),
        mapToEditPanelValues: (config) => new Map([
          ['quiz_title', config.quiz_title || defaultConfig.quiz_title],
          ['quiz_subtitle', config.quiz_subtitle || defaultConfig.quiz_subtitle],
          ['start_button_text', config.start_button_text || defaultConfig.start_button_text],
          ['submit_button_text', config.submit_button_text || defaultConfig.submit_button_text],
          ['results_title', config.results_title || defaultConfig.results_title]
        ])
      });
    }

    render();
  </script>
 <script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'9a8941c72446d018',t:'MTc2NDgyOTg0Ni4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script></body>
</html>
