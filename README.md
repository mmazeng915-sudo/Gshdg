/* Stage 1 behaviors (محسّن tolerant match) */

// دالة للتطبيع: تشيل مسافات زايدة، تطلع الحروف الصغيرة، وتبدّل بعض الاختلافات الشائعة
function normalizeArabic(s){
  if(!s) return '';
  // remove tatweel and diacritics
  s = s.replace(/ـ/g,'').normalize('NFC');
  // remove Arabic diacritics (tashkeel)
  s = s.replace(/[\u0610-\u061A\u064B-\u065F\u06D6-\u06ED]/g,'');
  // replace different forms of alef and hamza to a simple form
  s = s.replace(/[إأآا]/g,'ا');
  s = s.replace(/[يى]/g,'ي');
  s = s.replace(/ة/g,'ه'); // بنخليها تتطابق أحيانا
  s = s.replace(/\s+/g,' '); // collapse spaces
  s = s.trim();
  return s;
}

// دالة بسيطة تفحص إذا كانت المدخلة تشبه التريجر "ياواد يادقدق"
function isMamaCall(raw){
  const s = normalizeArabic(raw);
  if(!s) return false;
  // بعض أنماط ممكن يستقبلها: تحتوي على ياواد و/أو دقدق أو يادقدق
  // نتحقق بأن السلسلة فيها كلمات مفتاحية
  const hasYa = /يا/.test(s); // وجود يا
  const hasWad = /واد|وادد|وادو|وادِ?/.test(s);
  const hasDaq = /دقدق|دق دق|دق\d?|دقدق؟|دق/.test(s);
  // قبول لو فيها يا + (واد أو دقدق) أو كلها
  return (hasYa && (hasWad || hasDaq)) || /ياواد\s*د?د?قدق/.test(s);
}

// حدث زر "اسمع ماما"
hear1.addEventListener('click', ()=> {
  appendLog(log1, 'ماما: ياواد يادقدق', 'bot');
  sayArabic('ياواد يادقدق', {rate:0.98, pitch:0.9});
});

// حدث زر الرد — الآن يتحقق بتسامح أكبر
btn1.addEventListener('click', ()=> {
  const valRaw = input1.value || '';
  const val = valRaw.trim();
  if(!val){
    appendLog(log1, 'اكتب حاجة عشان ترد يا بطل', 'bot');
    return;
  }
  appendLog(log1, val, 'user');

  // نتحقّق بالنسخة المطهّرة
  if(isMamaCall(val)){
    appendLog(log1, 'نعم ياماما — نجمة كسبت! ⭐', 'bot');
    sayArabic('نعم يا ماما', {rate:1.1, pitch:1});
    stars += 1;
    setTimeout(()=> goToStage(2), 900);
    return;
  }

  // لو المدخل يبدو كونه رد (مثلاً "نعم يا ماما") — نرد عليه ونشرح
  const normalized = normalizeArabic(val);
  if(/نعم|حاضر|ايوه|ايوه|ايوه/.test(normalized)){
    appendLog(log1, 'تمام — بس اكتب "ياواد يادقدق" الأول عشان أختبرك 😉', 'bot');
    sayArabic('اكتب ياواد يادقدق الاول', {rate:1, pitch:0.95});
    return;
  }

  // غير كده
  appendLog(log1, 'ماما مش مرتاحة… حاول تاني (اكتب: ياواد يادقدق)', 'bot');
  sayArabic('مش سامع يا ماما', {rate:1, pitch:0.9});
});
