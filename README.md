import React, { useState, useEffect, useMemo } from 'react';
import { Phone, CheckCircle, Clock, XCircle, User, Calendar, FileText, Copy, BarChart2, Filter, ArrowRight, Save, MessageSquare, Globe, ChevronRight, Sparkles, Send, RefreshCw, Mail, ClipboardCopy } from 'lucide-react';

// --- API CONFIGURATION ---
// AIzaSyC_tS5pMOnrYCNrVJNP2F7Svz4gBX_5rLM""; 

// --- REAL LEADS DATA (Imported from CSV) ---
const INITIAL_LEADS = [
  { id: 1, firstName: "Earl", lastName: "Crawford", company: "Alameda County Juvenile Hall/Court", phone: "(510) 670-7609", email: "ecrawford@acoe.org", status: "In Progress", notes: "CSV Note: Yes called and spoke", lastContact: "2023-11-15", timezone: "PST", officeHours: "8:00 AM - 4:00 PM" },
  { id: 2, firstName: "Monica", lastName: "Vaughan", company: "Alameda County Special Education", phone: "(510) 723-3857", email: "mvaughan@acoe.org", status: "Follow Up", notes: "CSV Note: No answer", lastContact: "2023-11-15", timezone: "PST", officeHours: "8:00 AM - 4:00 PM" },
  { id: 3, firstName: "Daniel", lastName: "Zarazua", company: "Alternatives in Action", phone: "(510) 748-4314", email: "dzarazua@alternativesinaction.org", status: "Follow Up", notes: "CSV Note: No answer", lastContact: "2023-11-15", timezone: "PST", officeHours: "8:00 AM - 4:00 PM" },
  { id: 4, firstName: "David", lastName: "Diehl", company: "STRIVE Academy", phone: "(559) 898-6720", email: "david.diehl@selmausd.org", status: "New", notes: "District: Selma Unified", lastContact: null, timezone: "PST", officeHours: "7:30 AM - 3:30 PM" },
  { id: 5, firstName: "Sabrina", lastName: "Green", company: "Terry Elementary", phone: "(559) 898-6710", email: "sabrina.green@selmausd.org", status: "New", notes: "District: Selma Unified", lastContact: null, timezone: "PST", officeHours: "8:00 AM - 3:30 PM" },
  { id: 6, firstName: "Beth", lastName: "Doyle", company: "Theodore Roosevelt Elementary", phone: "(559) 898-6700", email: "beth.doyle@selmausd.org", status: "New", notes: "District: Selma Unified", lastContact: null, timezone: "PST", officeHours: "8:00 AM - 3:30 PM" },
  { id: 7, firstName: "Veronica", lastName: "Aguayo", company: "Woodrow Wilson Elementary", phone: "(559) 898-6670", email: "veronica.aguayo@selmausd.org", status: "New", notes: "District: Selma Unified", lastContact: null, timezone: "PST", officeHours: "8:00 AM - 3:30 PM" },
];

// --- GEMINI API HELPER ---
async function generateGeminiContent(prompt, systemInstruction = "") {
  if (!apiKey) return "⚠️ AI features require an API Key. Please add it to the code."; 
  
  try {
    const response = await fetch(
      `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`,
      {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          contents: [{ parts: [{ text: prompt }] }],
          systemInstruction: { parts: [{ text: systemInstruction }] }
        }),
      }
    );
    const data = await response.json();
    return data.candidates?.[0]?.content?.parts?.[0]?.text || "Sorry, I couldn't generate a response right now.";
  } catch (error) {
    console.error("AI Error:", error);
    return "Error connecting to AI service.";
  }
}

// --- COMPONENTS ---

const Header = ({ view, setView }) => (
  <header className="bg-white border-b border-gray-200 sticky top-0 z-50 shadow-sm">
    <div className="container mx-auto px-4 h-20 flex items-center justify-between">
      <div className="flex flex-col justify-center cursor-pointer group" onClick={() => setView('dashboard')}>
        <div className="flex items-center text-2xl font-bold tracking-tight">
          <span className="text-[#003B5C]">Language</span>
          <span className="text-[#0088BB] flex items-center relative">
            Pe
            <div className="relative mx-[1px]">
               <Globe className="w-5 h-5 text-[#003B5C] absolute top-0 left-0 opacity-20" />
               <Globe className="w-5 h-5 text-[#0088BB]" />
            </div>
            ple
          </span>
        </div>
        <span className="text-[10px] uppercase tracking-[0.25em] text-gray-400 ml-0.5 group-hover:text-[#0088BB] transition-colors">Connect. Communicate.</span>
      </div>
      <nav className="flex space-x-2">
        <button onClick={() => setView('dashboard')} className={`px-4 py-2 rounded-lg text-sm font-bold transition-all ${view === 'dashboard' ? 'bg-[#003B5C] text-white shadow-md transform scale-105' : 'text-gray-600 hover:bg-gray-50'}`}>Dashboard</button>
        <button onClick={() => setView('leads')} className={`px-4 py-2 rounded-lg text-sm font-bold transition-all ${view === 'leads' ? 'bg-[#003B5C] text-white shadow-md transform scale-105' : 'text-gray-600 hover:bg-gray-50'}`}>All Leads</button>
      </nav>
    </div>
  </header>
);

const StatCard = ({ title, value, icon: Icon, color }) => (
  <div className="bg-white p-6 rounded-xl shadow-[0_2px_10px_rgba(0,0,0,0.05)] border border-gray-100 flex items-center space-x-4 hover:-translate-y-1 transition-transform duration-300">
    <div className={`p-4 rounded-full ${color} bg-opacity-10`}>
      <Icon className={`w-6 h-6 ${color.replace('bg-', 'text-')}`} />
    </div>
    <div>
      <p className="text-xs text-gray-500 font-bold uppercase tracking-wider">{title}</p>
      <h3 className="text-3xl font-extrabold text-gray-800 mt-1">{value}</h3>
    </div>
  </div>
);

const CallScript = ({ lead }) => {
  const [activeTab, setActiveTab] = useState('intro');
  const [customObjection, setCustomObjection] = useState('');
  const [aiResponse, setAiResponse] = useState('');
  const [isLoadingAi, setIsLoadingAi] = useState(false);

  const handleAiObjection = async () => {
    if (!customObjection) return;
    setIsLoadingAi(true);
    const prompt = `The prospect just said: "${customObjection}". Give me a 2 sentence professional rebuttal for a salesperson at 'Language People', a translation agency. Focus on liability protection, certified quality, and 24/7 availability.`;
    const response = await generateGeminiContent(prompt, "You are an expert sales coach specializing in B2B language services.");
    setAiResponse(response);
    setIsLoadingAi(false);
  };

  const scripts = {
    intro: (
      <div className="space-y-4 animate-fade-in-up">
        <div className="bg-[#F0F9FC] p-6 rounded-xl border-l-4 border-[#0088BB] shadow-sm">
          <p className="text-xl text-[#003B5C] font-bold">"Hi, is this {lead.firstName}?"</p>
        </div>
        <div className="bg-white p-6 rounded-xl border border-gray-200 shadow-sm">
          <p className="text-gray-700 leading-relaxed text-lg">
            "My name is <strong>[Your Name]</strong> with <span className="text-[#003B5C] font-bold">Language People</span>. <br/><br/>
            We help organizations like <strong>{lead.company}</strong> bridge language barriers with professional interpretation."
          </p>
        </div>
      </div>
    ),
    value: (
      <div className="space-y-4 animate-fade-in-up">
        <div className="bg-white p-6 rounded-xl border border-gray-200 shadow-sm">
          <p className="text-gray-700 leading-relaxed text-lg mb-4">"I noticed you work with [specific population], and we specialize in ensuring effective communication through:"</p>
          <ul className="space-y-3">
            <li className="flex items-center text-gray-700 bg-slate-50 p-3 rounded-lg"><CheckCircle className="w-5 h-5 text-[#0088BB] mr-3"/> Video Remote Interpretation</li>
            <li className="flex items-center text-gray-700 bg-slate-50 p-3 rounded-lg"><CheckCircle className="w-5 h-5 text-[#0088BB] mr-3"/> Face-to-Face Interpreters</li>
            <li className="flex items-center text-gray-700 bg-slate-50 p-3 rounded-lg"><CheckCircle className="w-5 h-5 text-[#0088BB] mr-3"/> Document Translation</li>
          </ul>
        </div>
      </div>
    ),
    objection: (
      <div className="space-y-4 animate-fade-in-up">
        <div className="p-5 bg-orange-50 border-l-4 border-orange-400 rounded-r-xl shadow-sm">
          <strong className="block text-orange-900 text-xs font-bold uppercase tracking-wider mb-2">"We already have a provider"</strong>
          <p className="text-gray-800 italic">"That's great to hear. Many of our clients use us as a backup for when their primary vendor can't fill a request. Would you be open to seeing our rate sheet just for comparison?"</p>
        </div>
        <div className="p-5 bg-orange-50 border-l-4 border-orange-400 rounded-r-xl shadow-sm">
          <strong className="block text-orange-900 text-xs font-bold uppercase tracking-wider mb-2">"We use Google Translate"</strong>
          <p className="text-gray-800 italic">"I understand that's convenient, but for liability and accuracy—especially in professional settings—a certified human interpreter is often required. We can offer a free trial to show the difference."</p>
        </div>
      </div>
    ),
    voicemail: (
      <div className="bg-slate-50 p-6 rounded-xl border-2 border-dashed border-gray-300 animate-fade-in-up">
        <p className="text-gray-700 leading-relaxed font-mono text-sm">"Hi {lead.firstName}, this is [Your Name] with Language People. I'm calling to share how we can help {lead.company} reduce costs on interpretation services. I'll try you again next week, or you can reach me at (800) 894-2345."</p>
      </div>
    ),
    "AI Help ✨": (
      <div className="space-y-4 animate-fade-in-up">
        <div className="bg-indigo-50 p-5 rounded-xl border border-indigo-100">
          <label className="block text-sm font-bold text-indigo-900 mb-2">What did the prospect just say?</label>
          <div className="flex gap-2">
            <input 
              type="text" 
              value={customObjection}
              onChange={(e) => setCustomObjection(e.target.value)}
              placeholder="e.g., 'We don't have budget right now'"
              className="flex-1 border border-indigo-200 rounded-lg p-2 focus:ring-2 focus:ring-indigo-500 outline-none text-sm"
              onKeyDown={(e) => e.key === 'Enter' && handleAiObjection()}
            />
            <button 
              onClick={handleAiObjection}
              disabled={isLoadingAi}
              className="bg-indigo-600 text-white px-4 py-2 rounded-lg hover:bg-indigo-700 transition-colors disabled:opacity-50"
            >
              {isLoadingAi ? <RefreshCw className="w-5 h-5 animate-spin"/> : <Send className="w-5 h-5"/>}
            </button>
          </div>
        </div>
        {aiResponse && (
          <div className="bg-white p-5 rounded-xl border-l-4 border-indigo-500 shadow-sm">
            <strong className="flex items-center text-indigo-600 text-xs font-bold uppercase tracking-wider mb-2">
              <Sparkles className="w-3 h-3 mr-1" /> AI Suggested Rebuttal
            </strong>
            <p className="text-gray-800 italic text-lg">"{aiResponse}"</p>
          </div>
        )}
         {!aiResponse && !isLoadingAi && (
          <div className="text-center text-gray-400 text-sm py-8">
            <Sparkles className="w-8 h-8 mx-auto mb-2 opacity-20" />
            <p>Type what the client said above, and I'll generate a response instantly.</p>
          </div>
        )}
      </div>
    )
  };

  return (
    <div className="bg-white rounded-xl shadow-md border border-gray-200 h-full flex flex-col overflow-hidden">
      <div className="bg-[#003B5C] text-white px-6 py-4 flex items-center justify-between">
         <div className="flex items-center">
            <FileText className="w-5 h-5 mr-2 text-[#0088BB]"/>
            <span className="font-bold tracking-wide uppercase text-sm">Live Script Assistant</span>
         </div>
      </div>
      <div className="bg-gray-50 p-2 flex space-x-1 overflow-x-auto border-b border-gray-200">
        {Object.keys(scripts).map(key => (
          <button
            key={key}
            onClick={() => setActiveTab(key)}
            className={`flex-1 px-2 py-2 text-xs font-bold uppercase tracking-wide rounded-lg transition-all whitespace-nowrap ${activeTab === key ? 'bg-white text-[#003B5C] shadow-md transform scale-[1.02] border border-gray-100' : 'text-gray-500 hover:bg-gray-200'} ${key.includes('✨') ? 'text-indigo-600' : ''}`}
          >
            {key}
          </button>
        ))}
      </div>
      <div className="p-6 flex-grow overflow-y-auto bg-white">
        {scripts[activeTab]}
      </div>
    </div>
  );
};

const LeadCard = ({ lead, onCall }) => (
  <div className="bg-white p-5 rounded-xl shadow-sm border border-gray-200 hover:shadow-lg transition-all hover:border-[#0088BB] flex justify-between items-center group cursor-pointer" onClick={() => onCall(lead)}>
    <div>
      <h3 className="font-bold text-gray-800 text-lg group-hover:text-[#003B5C] transition-colors">{lead.firstName} {lead.lastName}</h3>
      <p className="text-sm text-gray-500 font-medium">{lead.company}</p>
      <div className="flex items-center space-x-2 mt-3">
        <span className={`px-2 py-1 rounded-md text-[10px] font-bold uppercase tracking-wide border 
          ${lead.status === 'New' ? 'bg-blue-50 text-blue-700 border-blue-100' : 
            lead.status === 'Closed' ? 'bg-green-50 text-green-700 border-green-100' : 
            lead.status === 'Lost' ? 'bg-red-50 text-red-700 border-red-100' : 
            lead.status === 'In Progress' ? 'bg-yellow-50 text-yellow-700 border-yellow-100' : 'bg-gray-100 text-gray-600 border-gray-200'}`}>
          {lead.status}
        </span>
        {lead.lastContact && <span className="text-[10px] text-gray-400 flex items-center font-medium"><Clock className="w-3 h-3 mr-1"/> {lead.lastContact}</span>}
      </div>
    </div>
    <button className="w-12 h-12 rounded-full bg-slate-100 text-[#0088BB] flex items-center justify-center group-hover:bg-[#0088BB] group-hover:text-white transition-all duration-300 shadow-sm" title="Start Call">
      <Phone className="w-5 h-5" />
    </button>
  </div>
);

// --- MAIN APP COMPONENT ---

export default function SalesAssistantApp() {
  const [leads, setLeads] = useState(() => {
    const saved = localStorage.getItem('lp_leads_live'); // Updated Key for Fresh Data
    return saved ? JSON.parse(saved) : INITIAL_LEADS;
  });
  
  const [view, setView] = useState('dashboard'); 
  const [activeLead, setActiveLead] = useState(null);
  const [copyFeedback, setCopyFeedback] = useState('');
  const [emailCopyFeedback, setEmailCopyFeedback] = useState('');
  
  // Email Draft State
  const [isGeneratingEmail, setIsGeneratingEmail] = useState(false);
  const [draftEmail, setDraftEmail] = useState('');

  useEffect(() => {
    localStorage.setItem('lp_leads_live', JSON.stringify(leads)); // Updated Key for Fresh Data
  }, [leads]);

  const stats = useMemo(() => {
    const today = new Date().toISOString().split('T')[0];
    return {
      total: leads.length,
      calledToday: leads.filter(l => l.lastContact === today).length,
      new: leads.filter(l => l.status === 'New').length,
      closed: leads.filter(l => l.status === 'Closed').length,
      followUp: leads.filter(l => l.status === 'Follow Up').length,
    };
  }, [leads]);

  const handleStartCall = (lead) => {
    setActiveLead(lead);
    setView('call');
    setDraftEmail(''); // Reset draft
  };

  const handleCopyNumber = (number) => {
    navigator.clipboard.writeText(number);
    setCopyFeedback('Copied!');
    setTimeout(() => setCopyFeedback(''), 2000);
  };

  const updateLeadStatus = (leadId, newStatus, noteText) => {
    const today = new Date().toISOString().split('T')[0];
    setLeads(leads.map(l => {
      if (l.id === leadId) {
        return { 
          ...l, 
          status: newStatus, 
          lastContact: today,
          notes: noteText ? `${l.notes} \n[${today}]: ${noteText}` : l.notes
        };
      }
      return l;
    }));
    setView('leads'); 
  };

  const handleDraftEmail = async () => {
    const currentNotes = document.getElementById('callNotes')?.value || "No specific notes.";
    if (!activeLead) return;
    
    setIsGeneratingEmail(true);
    const prompt = `Write a polite, professional sales follow-up email to ${activeLead.firstName} ${activeLead.lastName} at ${activeLead.company}. 
    My notes from the call I just finished are: "${currentNotes}". 
    The email should come from "The Language People Team". 
    Keep it under 100 words. Encourage the next step based on the notes. Do not include subject line or placeholders.`;
    
    const emailContent = await generateGeminiContent(prompt, "You are a professional sales copywriter.");
    setDraftEmail(emailContent);
    setIsGeneratingEmail(false);
  };

  const handleCopyEmail = () => {
    navigator.clipboard.writeText(draftEmail);
    setEmailCopyFeedback('Copied for Outlook!');
    setTimeout(() => setEmailCopyFeedback(''), 3000);
  };

  return (
    <div className="min-h-screen bg-[#F8FAFC] text-slate-800 font-sans selection:bg-[#0088BB] selection:text-white">
      <Header view={view} setView={setView} />

      <main className="container mx-auto px-4 py-8 max-w-6xl">
        {/* DASHBOARD VIEW */}
        {view === 'dashboard' && (
          <div className="space-y-8 animate-fade-in">
            <div className="flex flex-col md:flex-row justify-between items-end border-b border-gray-200 pb-8 gap-4">
              <div>
                <h2 className="text-4xl font-extrabold text-[#003B5C] tracking-tight">Good Afternoon, Sales Team</h2>
                <p className="text-gray-500 mt-2 text-lg">Ready to connect the world today?</p>
              </div>
              <button 
                onClick={() => {
                  const nextLead = leads.find(l => l.status === 'New');
                  if(nextLead) handleStartCall(nextLead);
                  else setView('leads');
                }}
                className="bg-gradient-to-r from-[#003B5C] to-[#0088BB] hover:from-[#002a42] hover:to-[#0077a3] text-white px-8 py-4 rounded-full shadow-lg flex items-center space-x-3 transition-all transform hover:scale-105 font-bold text-lg w-full md:w-auto justify-center"
              >
                <Phone className="w-6 h-6 animate-pulse" />
                <span>Start Power Hour</span>
              </button>
            </div>

            {/* Updated Stats Grid with 'Calls Today' and 'Needs Callback' */}
            <div className="grid grid-cols-1 md:grid-cols-4 gap-4">
              <StatCard title="Calls Made Today" value={stats.calledToday} icon={Phone} color="bg-blue-600 text-blue-600" />
              <StatCard title="Needs Call Back" value={stats.followUp} icon={Clock} color="bg-amber-500 text-amber-500" />
              <StatCard title="Closed Deals" value={stats.closed} icon={BarChart2} color="bg-green-500 text-green-500" />
              <StatCard title="New Leads" value={stats.new} icon={User} color="bg-purple-500 text-purple-500" />
            </div>

            <div className="bg-white rounded-2xl shadow-sm border border-gray-200 overflow-hidden">
              <div className="px-8 py-6 border-b border-gray-100 flex justify-between items-center bg-slate-50">
                <h3 className="font-bold text-xl text-[#003B5C]">Recommended Next Calls</h3>
                <button onClick={() => setView('leads')} className="text-[#0088BB] text-sm font-bold hover:underline flex items-center group">
                  View All <ChevronRight className="w-4 h-4 ml-1 group-hover:translate-x-1 transition-transform" />
                </button>
              </div>
              <div className="divide-y divide-gray-100">
                {leads.filter(l => ['New', 'Follow Up'].includes(l.status)).slice(0, 5).map(lead => (
                  <div key={lead.id} className="px-8 py-5 flex items-center justify-between hover:bg-blue-50 transition-colors group cursor-pointer" onClick={() => handleStartCall(lead)}>
                    <div className="flex items-center space-x-4">
                      <div className={`w-3 h-3 rounded-full ring-2 ring-offset-2 ${lead.status === 'New' ? 'bg-blue-500 ring-blue-200' : 'bg-amber-500 ring-amber-200'}`}></div>
                      <div>
                        <p className="font-bold text-gray-800 text-lg">{lead.firstName} {lead.lastName}</p>
                        <p className="text-sm text-gray-500">{lead.company} • <span className="font-medium text-gray-400">{lead.timezone}</span></p>
                      </div>
                    </div>
                    <button 
                      onClick={(e) => { e.stopPropagation(); handleStartCall(lead); }}
                      className="text-[#0088BB] bg-white border border-[#0088BB] hover:bg-[#0088BB] hover:text-white px-6 py-2 rounded-full text-sm font-bold transition-all shadow-sm"
                    >
                      Call Now
                    </button>
                  </div>
                ))}
              </div>
            </div>
          </div>
        )}

        {/* LEADS LIST VIEW */}
        {view === 'leads' && (
          <div className="animate-fade-in">
            <div className="flex justify-between items-center mb-8">
              <h2 className="text-3xl font-bold text-[#003B5C]">All Leads</h2>
              <div className="flex space-x-3">
                <button className="flex items-center space-x-2 px-5 py-2.5 border border-gray-200 rounded-lg bg-white text-gray-600 hover:bg-gray-50 font-medium shadow-sm">
                  <Filter className="w-4 h-4" />
                  <span>Filter</span>
                </button>
                <button className="flex items-center space-x-2 px-5 py-2.5 border border-gray-200 rounded-lg bg-white text-gray-600 hover:bg-gray-50 font-medium shadow-sm">
                  <FileText className="w-4 h-4" />
                  <span>Import CSV</span>
                </button>
              </div>
            </div>
            <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
              {leads.map(lead => (
                <LeadCard key={lead.id} lead={lead} onCall={handleStartCall} />
              ))}
            </div>
          </div>
        )}

        {/* ACTIVE CALL VIEW */}
        {view === 'call' && activeLead && (
          <div className="animate-fade-in h-[calc(100vh-140px)] flex flex-col lg:flex-row gap-6">
            
            {/* Left Panel: Lead Info & Script */}
            <div className="lg:w-2/3 flex flex-col gap-6">
              <div className="bg-white p-8 rounded-2xl shadow-sm border border-gray-200 flex justify-between items-start relative overflow-hidden">
                <div className="absolute top-0 left-0 w-2 h-full bg-[#0088BB]"></div>
                <div>
                  <div className="flex items-center space-x-4 mb-2">
                    <h2 className="text-4xl font-bold text-[#003B5C]">{activeLead.firstName} {activeLead.lastName}</h2>
                    <span className={`px-3 py-1 rounded-full text-xs font-bold uppercase tracking-wider border ${activeLead.status === 'New' ? 'bg-blue-50 text-blue-700 border-blue-200' : 'bg-gray-100 text-gray-600 border-gray-200'}`}>
                      {activeLead.status}
                    </span>
                  </div>
                  <p className="text-xl text-gray-500 mb-6 font-medium">{activeLead.company}</p>
                  <div className="flex items-center space-x-4">
                    <a 
                      href={`tel:${activeLead.phone}`}
                      className="bg-[#22c55e] hover:bg-[#16a34a] text-white px-8 py-4 rounded-xl shadow-lg flex items-center space-x-3 transition-all hover:-translate-y-0.5 font-bold text-xl"
                    >
                      <Phone className="w-6 h-6" />
                      <span>Dial {activeLead.phone}</span>
                    </a>
                    <button 
                      onClick={() => handleCopyNumber(activeLead.phone)}
                      className="border-2 border-gray-200 bg-white text-gray-500 px-5 py-4 rounded-xl hover:border-[#0088BB] hover:text-[#0088BB] transition-all relative group"
                      title="Copy to clipboard"
                    >
                      <Copy className="w-6 h-6" />
                      {copyFeedback && (
                        <span className="absolute -top-10 left-1/2 transform -translate-x-1/2 bg-[#003B5C] text-white text-xs font-bold py-1 px-3 rounded shadow-lg">
                          {copyFeedback}
                        </span>
                      )}
                    </button>
                  </div>
                </div>
                <div className="text-right text-sm text-gray-500 bg-slate-50 p-5 rounded-xl border border-slate-100 hidden md:block">
                  <p className="mb-2">Timezone: <strong className="text-gray-800">{activeLead.timezone}</strong></p>
                  <p className="mb-2">Office Hours: <strong className="text-gray-800">{activeLead.officeHours}</strong></p>
                  <p>Email: <a href={`mailto:${activeLead.email}`} className="text-[#0088BB] hover:underline font-bold">{activeLead.email}</a></p>
                </div>
              </div>
              <div className="flex-grow">
                <CallScript lead={activeLead} />
              </div>
            </div>

            {/* Right Panel: Action & Notes */}
            <div className="lg:w-1/3 bg-white rounded-2xl shadow-sm border border-gray-200 flex flex-col overflow-hidden">
              <div className="p-5 border-b border-gray-200 bg-[#003B5C] text-white">
                <h3 className="font-bold flex items-center text-lg">
                  <MessageSquare className="w-5 h-5 mr-2 text-[#0088BB]" />
                  Call Outcome & Notes
                </h3>
              </div>
              
              <div className="p-6 flex-grow flex flex-col space-y-4">
                {activeLead.notes && (
                  <div className="bg-amber-50 p-4 rounded-xl border border-amber-100 text-sm text-gray-700 max-h-32 overflow-y-auto shadow-inner">
                    <strong className="block text-amber-900 mb-1 text-xs uppercase tracking-wide">Previous History</strong> 
                    {activeLead.notes}
                  </div>
                )}

                <div className="flex-grow flex flex-col">
                  <div className="flex justify-between items-center mb-2">
                     <label className="block text-sm font-bold text-gray-700">Current Call Notes</label>
                     {!draftEmail && (
                        <button 
                          onClick={handleDraftEmail}
                          disabled={isGeneratingEmail}
                          className="text-xs flex items-center text-indigo-600 hover:text-indigo-800 font-medium transition-colors disabled:opacity-50 bg-indigo-50 px-2 py-1 rounded-md border border-indigo-100"
                        >
                          {isGeneratingEmail ? <RefreshCw className="w-3 h-3 animate-spin mr-1"/> : <Sparkles className="w-3 h-3 mr-1"/>}
                          {isGeneratingEmail ? 'Writing...' : 'Generate Email Draft'}
                        </button>
                     )}
                  </div>
                  
                  {!draftEmail ? (
                    <textarea 
                      id="callNotes"
                      className="w-full flex-grow border-2 border-gray-200 rounded-xl p-4 focus:ring-0 focus:border-[#0088BB] outline-none text-base transition-colors resize-none bg-slate-50"
                      placeholder="Type outcome here, then click Generate Email to create a follow-up..."
                    ></textarea>
                  ) : (
                    <div className="flex-grow flex flex-col border-2 border-indigo-100 bg-indigo-50 rounded-xl p-4 overflow-hidden relative group">
                      <p className="text-sm text-indigo-900 whitespace-pre-wrap font-sans mb-8 overflow-y-auto flex-grow">{draftEmail}</p>
                      
                      {/* Outlook Copy Section */}
                      <div className="absolute bottom-0 left-0 right-0 p-2 bg-indigo-100/90 border-t border-indigo-200 flex justify-between items-center">
                         <button 
                            onClick={handleCopyEmail} 
                            className="flex items-center justify-center bg-blue-600 hover:bg-blue-700 text-white text-xs font-bold py-2 px-4 rounded shadow-sm flex-grow mr-2 transition-colors"
                         >
                            <ClipboardCopy className="w-4 h-4 mr-2" />
                            {emailCopyFeedback || "Copy for Outlook"}
                         </button>
                         <button onClick={() => setDraftEmail('')} className="text-gray-500 hover:text-red-600 p-2" title="Discard">
                           <XCircle className="w-5 h-5"/>
                         </button>
                      </div>
                    </div>
                  )}
                </div>

                <div className="grid grid-cols-2 gap-3">
                  <button 
                    onClick={() => updateLeadStatus(activeLead.id, 'Follow Up', document.getElementById('callNotes')?.value + " (No Answer)")}
                    className="bg-gray-100 hover:bg-gray-200 text-gray-600 py-3 rounded-lg font-bold text-sm transition-colors"
                  >
                    No Answer
                  </button>
                  <button 
                    onClick={() => updateLeadStatus(activeLead.id, 'Follow Up', document.getElementById('callNotes')?.value + " (Left VM)")}
                    className="bg-orange-100 hover:bg-orange-200 text-orange-700 py-3 rounded-lg font-bold text-sm transition-colors"
                  >
                    Left Voicemail
                  </button>
                  <button 
                    onClick={() => updateLeadStatus(activeLead.id, 'Lost', document.getElementById('callNotes')?.value + " (Not Interested)")}
                    className="bg-red-50 hover:bg-red-100 text-red-700 py-3 rounded-lg font-bold text-sm transition-colors"
                  >
                    Not Interested
                  </button>
                  <button 
                    onClick={() => updateLeadStatus(activeLead.id, 'In Progress', document.getElementById('callNotes')?.value)}
                    className="bg-blue-100 hover:bg-blue-200 text-blue-700 py-3 rounded-lg font-bold text-sm transition-colors"
                  >
                    Interested
                  </button>
                  <button 
                    onClick={() => updateLeadStatus(activeLead.id, 'Closed', document.getElementById('callNotes')?.value)}
                    className="col-span-2 bg-green-600 hover:bg-green-700 text-white py-3 rounded-lg font-bold text-sm transition-colors shadow-md"
                  >
                    Sale Closed!
                  </button>
                </div>
                
                <button onClick={() => setView('leads')} className="w-full pt-2 text-gray-400 text-xs hover:text-gray-600 font-medium text-center">Cancel Call</button>
              </div>
            </div>
          </div>
        )}
      </main>
    </div>
  );
}
