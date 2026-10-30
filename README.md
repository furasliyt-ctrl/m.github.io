<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>منصة مِران التعليمية الشاملة</title>
<script src="https://cdn.tailwindcss.com"></script>
<link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;600;700&display=swap" rel="stylesheet">
<style>
body { font-family:'Cairo',sans-serif; }
</style>
</head>
<body class="bg-slate-50 text-slate-800 min-h-screen p-4 md:p-8">

<main class="max-w-4xl mx-auto">

<header class="text-center mb-8">
<h1 class="text-3xl font-bold text-indigo-700 mb-2">
منصة مِران التعليمية الشاملة
</h1>
<p class="text-slate-600">
المساعد التعليمي للمناهج العراقية – المرحلة المتوسطة
</p>
</header>

<div class="flex flex-wrap justify-center gap-3 mb-6">
<button onclick="setMode('teacher')" id="btn-teacher"
class="px-5 py-3 rounded-xl font-bold shadow-md bg-indigo-600 text-white">
👨‍🏫 وضع المعلم
</button>

<button onclick="setMode('student')" id="btn-student"
class="px-5 py-3 rounded-xl font-bold shadow-md bg-white text-slate-700 border">
🎓 وضع الطالب
</button>
</div>

<section class="bg-white rounded-2xl shadow-xl p-6 md:p-8 mb-8">

<!-- حقل إدخال مفتاح Google AI Studio -->
<div class="mb-5 p-4 bg-slate-50 rounded-xl border border-slate-200">
<label class="block font-semibold mb-1 text-slate-700 text-sm">🔑 مفتاح Google AI Studio الخاص بك:</label>
<input id="apiKeyInput" type="password"
placeholder="ألصق مفتاح API هنا..."
class="w-full p-2.5 rounded-lg border text-sm font-mono focus:outline-none focus:ring-2 focus:ring-indigo-500">
</div>

<div class="mb-4">
<label class="block font-semibold mb-2">الصف الدراسي:</label>
<select id="gradeLevel" class="w-full p-3 rounded-xl border">
<option>الثالث المتوسط</option>
<option>الثاني المتوسط</option>
<option>الأول المتوسط</option>
</select>
</div>

<div class="mb-4">
<label class="block font-semibold mb-2">المادة الدراسية:</label>
<select id="subject" class="w-full p-3 rounded-xl border">
<option>الاجتماعيات (تاريخ، جغرافيا، وطنية)</option>
<option>الرياضيات</option>
<option>اللغة العربية</option>
<option>اللغة الإنجليزية</option>
<option>العلوم العامة</option>
<option>التربية الإسلامية</option>
<option>الحاسوب</option>
</select>
</div>

<div class="mb-4">
<label id="task-label" class="block font-semibold mb-2">
نوع المهمة المطلوبة:
</label>
<select id="taskType" class="w-full p-3 rounded-xl border"></select>
</div>

<div class="mb-6">
<label id="topic-label" class="block font-semibold mb-2">
عنوان الدرس أو موضوع الخطة المطلوبة:
</label>
<input id="topicInput" type="text"
placeholder="اكتب سؤالك أو موضوع الدرس هنا..."
class="w-full p-3 rounded-xl border">
</div>

<div class="grid grid-cols-1 md:grid-cols-2 gap-4">

<button onclick="handleGeneratePrompt()"
class="bg-emerald-600 hover:bg-emerald-700 text-white font-bold py-3.5 rounded-xl transition-all shadow-md">
✨ توليد البرومبت المنهجي
</button>

<button onclick="handleFetchAnswer()" id="answerButton"
class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-3.5 rounded-xl transition-all shadow-md">
🚀 الحصول على الإجابة المباشرة
</button>

</div>

</section>

<section id="outputContainer"
class="bg-slate-900 text-slate-100 rounded-2xl p-6 hidden mb-8">

<div class="flex justify-between items-center mb-3">
<h2 class="font-bold text-emerald-400">
البرومبت الجاهز للنسخ:
</h2>
<button onclick="copyPromptText()"
class="bg-slate-800 text-xs px-3 py-2 rounded-lg text-white">
📋 نسخ النص
</button>
</div>

<pre id="resultText"
class="whitespace-pre-wrap text-sm leading-relaxed text-slate-300 bg-slate-950 p-4 rounded-xl"></pre>

</section>

<section id="answerContainer"
class="bg-white rounded-2xl shadow-xl p-6 hidden">

<div class="flex justify-between items-center border-b pb-4 mb-4">
<h2 class="font-bold text-xl text-indigo-700">
📖 الإجابة التعليمية
</h2>
<span id="loadingBadge" class="hidden text-amber-700 text-xs animate-pulse font-semibold">
جارٍ إعداد الإجابة...
</span>
</div>

<div id="answerText"
class="text-slate-700 leading-relaxed whitespace-pre-wrap"></div>

</section>

</main>

<script>
const tasksData = {
teacher: [
{value:'lesson_plan',text:'📅 إعداد خطة تدريس سنوية أو يومية تفصيلية'},
{value:'booklet_content',text:'📖 تصميم محتوى ملزمة دراسية مبسطة'},
{value:'exam_maker',text:'📝 توليد أسئلة امتحانية شاملة'},
{value:'weekly_schedule',text:'🗓️ بناء جدول توزيع الحصص الأسبوعي'}
],
student: [
{value:'explain_simple',text:'💡 شرح الدرس ببساطة وتفصيل'},
{value:'study_schedule',text:'📅 تصميم جدول مذاكرة منظم'},
{value:'self_quiz',text:'🧠 اختبار ذاتي تفاعلي سريع'},
{value:'summary_table',text:'📊 تلخيص ومقارنة المفاهيم الصعبة'}
]
};

let currentMode='teacher';

function setMode(mode) {
currentMode=mode;

const active='px-5 py-3 rounded-xl font-bold shadow-md bg-indigo-600 text-white';
const inactive='px-5 py-3 rounded-xl font-bold shadow-md bg-white text-slate-700 border';

document.getElementById('btn-teacher').className=mode==='teacher'?active:inactive;
document.getElementById('btn-student').className=mode==='student'?active:inactive;

document.getElementById('topic-label').textContent=
mode==='teacher'
?'عنوان الدرس أو موضوع الخطة المطلوبة:'
:'الموضوع أو السؤال المراد الإجابة عنه:';

updateTasksDropdown();
}

function updateTasksDropdown() {
const select=document.getElementById('taskType');
select.replaceChildren();

tasksData[currentMode].forEach(task=>{
const option=document.createElement('option');
option.value=task.value;
option.textContent=task.text;
select.appendChild(option);
});
}

function getPromptContent() {
const grade=document.getElementById('gradeLevel').value;
const subject=document.getElementById('subject').value;
const topic=document.getElementById('topicInput').value.trim();
const select=document.getElementById('taskType');
const task=select.options[select.selectedIndex]?.text||'';

if(!topic)return null;

const role=currentMode==='teacher'
?`بصفتك خبيراً تربوياً في مناهج وزارة التربية العراقية الرسمية لـ (${grade}) لمادة (${subject})`
:`بصفتك معلماً خصوصياً صبوراً لمناهج العراق لـ (${grade}) في مادة (${subject})`;

const instruction=currentMode==='teacher'
?'اعتمد الكتاب المدرسي المقرر قدر الإمكان. إذا لم يتوفر نص الكتاب فلا تنسب معلومة إليه، واذكر بوضوح ما يحتاج إلى تحقق. استخدم أسلوباً مباشراً ومنظماً ومباشرة دون استخدام جداول.'
:'اشرح للطالب بلغة مبسطة ومباشرة. اعتمد الكتاب المدرسي المقرر قدر الإمكان، ولا تخترع نصوصاً أو أرقام صفحات. وضّح إذا كانت المعلومة تحتاج إلى تحقق ودون استخدام جداول.';

return `${role}:
المهمة: (${task}).
الموضوع/السؤال: (${topic}).

تعليمات صارمة: ${instruction}`;
}

function handleGeneratePrompt() {
const prompt=getPromptContent();

if(!prompt){
alert('يرجى كتابة الموضوع أو السؤال أولاً!');
return;
}

document.getElementById('resultText').textContent=prompt;
document.getElementById('outputContainer').classList.remove('hidden');
document.getElementById('answerContainer').classList.add('hidden');
document.getElementById('outputContainer').scrollIntoView({behavior:'smooth'});
}

async function handleFetchAnswer() {
const prompt=getPromptContent();
const apiKey = document.getElementById('apiKeyInput').value.trim();

if(!prompt){
alert('يرجى كتابة السؤال أو الموضوع أولاً!');
return;
}

if(!apiKey){
alert('يرجى إدخال مفتاح Google AI Studio في الحقل المخصص في الأعلى أولاً!');
document.getElementById('apiKeyInput').scrollIntoView({behavior:'smooth'});
return;
}

const container=document.getElementById('answerContainer');
const answer=document.getElementById('answerText');
const badge=document.getElementById('loadingBadge');
const button=document.getElementById('answerButton');

document.getElementById('outputContainer').classList.add('hidden');
container.classList.remove('hidden');
badge.classList.remove('hidden');
button.disabled=true;

answer.textContent='جارٍ الاتصال المباشر واستخراج الإجابة المعتمدة...';
container.scrollIntoView({behavior:'smooth'});

try {
// استخدام مسار الاتصال المباشر لنموذج gemini-2.5-flash
const endpoint = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=${apiKey}`;

const response = await fetch(endpoint, {
method: 'POST',
headers: {'Content-Type': 'application/json'},
body: JSON.stringify({
contents: [{ parts: [{ text: prompt }] }]
})
});

const data = await response.json();

if(!response.ok) {
throw new Error(data.error?.message || 'تعذر الاتصال بخدمة الذكاء الاصطناعي، تأكد من صلاحية المفتاح.');
}

if (data.candidates && data.candidates[0].content.parts[0].text) {
answer.textContent = data.candidates[0].content.parts[0].text;
} else {
throw new Error('لم تتوفر إجابة نصية صالحة من الاستجابة.');
}

} catch(error) {
answer.textContent = `تعذر جلب الإجابة: ${error.message}\n\nتأكد من صحة المفتاح أو اتصال الإنترنت.`;
} finally {
badge.classList.add('hidden');
button.disabled=false;
}
}

async function copyPromptText() {
const text=document.getElementById('resultText').textContent;

if(!text)return;

try {
await navigator.clipboard.writeText(text);
alert('تم نسخ البرومبت بنجاح!');
} catch {
alert('تعذر النسخ تلقائياً. يمكنك تحديد النص ونسخه يدوياً.');
}
}

window.addEventListener('DOMContentLoaded',updateTasksDropdown);
</script>

</body>
</html>
