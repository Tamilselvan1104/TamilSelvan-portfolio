import React, { useState, useEffect, useRef } from 'react';
import { Mail, Linkedin, Globe, Code, Award, Users, MessageSquareText, Lightbulb, GraduationCap, Sparkles, Github } from 'lucide-react';
import { initializeApp } from 'firebase/app';
import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from 'firebase/auth';
import { getFirestore } from 'firebase/firestore';

function App() {
  // Firebase setup - Mandatory global variables
  const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
  const firebaseConfig = typeof _firebase_config !== 'undefined' ? JSON.parse(_firebase_config) : {};
  const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

  const [db, setDb] = useState(null);
  const [auth, setAuth] = useState(null);
  const [userId, setUserId] = useState(null);
  const [isAuthReady, setIsAuthReady] = useState(false);

  // Portfolio data
  const portfolio = {
    name: "Tamilselvan J",
    title: "Electronics and Communication Engineering Student | IoT Enthusiast | Problem Solver",
    location: "Dharmapuri, Tamil Nadu, India",
    email: "tamilj996@gmail.com",
    linkedin: "https://www.linkedin.com/in/tamilselvan-j1104/",
    github: "https://github.com/Tamilselvan1104", // Changed from Githubwebsite to github for consistency
    about: "I'm a passionate 4th-year Electronics and Communication Engineering student with a strong interest in IoT, programming, and tech innovation. I enjoy solving real-world problems and contributing to impactful, tech-driven solutions.",
    technicalSkills: {
      languages: ["JAVA (Basics)", "C"],
      tools: ["Arduino IDE", "Thingspeak", "Tinkercad"],
      networking: ["Cisco Packet Tracer", "NS2", "Network Protocols"],
      iot: ["Sensor integration", "Data visualization", "Real-time acquisition"]
    },
    projects: [
      {
        title: "Low-Cost IoT-Based Weather Monitoring System",
        description: "Developed a real-time weather monitoring system using Thingspeak and integrated sensors for temperature, humidity, and pressure. Designed a web-based dashboard for remote weather tracking, especially useful for rural areas."
      },
      {
        title: "Line Follower Robot",
        description: "Built a robot capable of following a pre-defined path using IR sensors and Arduino – won 2nd prize in a robotics competition at Takshashila University."
      }
    ],
    achievements: [
      {
        title: "2nd Prize - Takshashila University Autonomous Robot Design Challenge",
        description: "Designed and built an autonomous robot that follows a predefined path using infrared sensors and adaptive navigation logic."
      }
    ],
    leadershipTeamwork: [
      "Led multiple IoT-based academic projects and hackathons.",
      "Organized and managed technical events and workshops at DRESTEIN (Annual Symposium).",
      "Collaborated with peers to design innovative solutions in automation and embedded systems."
    ],
    education: [
      {
        degree: "B.E. ECE",
        institution: "Saveetha Engineering College",
        details: "CGPA: 7.84 (2022-Present)"
      },
      {
        degree: "HSC",
        institution: "SVM Matric Hr. Sec. School",
        details: "79.6% (2021-2022)"
      },
      {
        degree: "SSLC",
        institution: "SVM Matric Hr. Sec. School",
        details: "94.6% (2019-2020)"
      }
    ],
    languages: [
      { name: "Tamil", proficiency: "Native proficiency" },
      { name: "English", proficiency: "Professional proficiency" }
    ]
  };

  // State to track which sections are currently visible for animation
  const [visibleSections, setVisibleSections] = useState({});
  // State to store elaborated project descriptions
  const [elaboratedDescriptions, setElaboratedDescriptions] = useState({});
  // State for loading indicator during LLM calls for projects
  const [projectLoadingStates, setProjectLoadingStates] = useState({});
  // State to store generated achievement impact statements
  const [achievementImpacts, setAchievementImpacts] = useState({});
  // State for loading indicator during LLM calls for achievements
  const [achievementLoadingStates, setAchievementLoadingStates] = useState({});
  // State for message box (e.g., error messages)
  const [message, setMessage] = useState({ text: '', type: '' });

  // Refs for each section to observe their intersection with the viewport
  const sectionRefs = {
    about: useRef(null),
    skills: useRef(null),
    projects: useRef(null),
    achievements: useRef(null),
    leadership: useRef(null),
    education: useRef(null),
    languages: useRef(null),
  };

  // Effect to set up IntersectionObserver for each section
  useEffect(() => {
    const observerOptions = {
      root: null, // viewport as the root
      rootMargin: '0px',
      threshold: 0.1, // Trigger when 10% of the section is visible
    };

    const observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          setVisibleSections(prev => ({ ...prev, [entry.target.id]: true }));
        }
      });
    }, observerOptions);

    // Observe each section ref
    Object.keys(sectionRefs).forEach(key => {
      if (sectionRefs[key].current) {
        observer.observe(sectionRefs[key].current);
      }
    });

    // Clean up observer on component unmount
    return () => {
      Object.keys(sectionRefs).forEach(key => {
        if (sectionRefs[key].current) {
          observer.unobserve(sectionRefs[key].current);
        }
      });
    };
  }, []);

  // Firebase Initialization and Authentication
  useEffect(() => {
    if (Object.keys(firebaseConfig).length > 0) {
      try {
        const app = initializeApp(firebaseConfig);
        const authInstance = getAuth(app);
        const firestoreInstance = getFirestore(app);
        setAuth(authInstance);
        setDb(firestoreInstance);

        const unsubscribe = onAuthStateChanged(authInstance, async (user) => {
          if (user) {
            setUserId(user.uid);
          } else {
            // Sign in anonymously if no initial token or user
            if (initialAuthToken) {
              try {
                await signInWithCustomToken(authInstance, initialAuthToken);
              } catch (error) {
                console.error("Error signing in with custom token:", error);
                await signInAnonymously(authInstance); // Fallback to anonymous
              }
            } else {
              await signInAnonymously(authInstance);
            }
          }
          setIsAuthReady(true); // Auth state is ready
        });
        return () => unsubscribe();
      } catch (error) {
        console.error("Error initializing Firebase:", error);
        setMessage({ text: 'Failed to initialize Firebase. Some features may not work.', type: 'error' });
      }
    } else {
      console.warn("Firebase config is empty. Firebase features will not be available.");
      setIsAuthReady(true); // Mark as ready even if not configured
    }
  }, [firebaseConfig, initialAuthToken]);

  // Function to apply animation classes based on visibility
  const getAnimationClasses = (sectionId) => {
    return visibleSections[sectionId] ? 'animate-fade-in-up' : 'opacity-0 translate-y-8';
  };

  /**
   * Calls the Gemini API to elaborate on a project description.
   * @param {string} projectTitle - The title of the project.
   * @param {string} currentDescription - The current description of the project.
   * @param {number} index - The index of the project in the array.
   */
  const elaborateProjectDescription = async (projectTitle, currentDescription, index) => {
    setProjectLoadingStates(prev => ({ ...prev, [index]: true })); // Set loading for specific project
    setMessage({ text: '', type: '' }); // Clear previous messages

    try {
      let chatHistory = [];
      const prompt = Elaborate on the following project description, providing more details and context. Focus on the technical aspects, challenges faced, and impact. Keep it concise but informative, around 100-150 words.\n\nProject Title: "${projectTitle}"\nOriginal Description: "${currentDescription}"\n\nElaborated Description:;
      chatHistory.push({ role: "user", parts: [{ text: prompt }] });

      const payload = { contents: chatHistory };
      const apiKey = ""; // Canvas will automatically provide the API key at runtime
      const apiUrl = https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey};

      const response = await fetch(apiUrl, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
      });

      if (!response.ok) {
        const errorData = await response.json();
        throw new Error(API error: ${response.status} - ${errorData.error?.message || response.statusText});
      }

      const result = await response.json();

      if (result.candidates && result.candidates.length > 0 &&
          result.candidates[0].content && result.candidates[0].content.parts &&
          result.candidates[0].content.parts.length > 0) {
        const text = result.candidates[0].content.parts[0].text;
        setElaboratedDescriptions(prev => ({ ...prev, [index]: text }));
        setMessage({ text: 'Project description elaborated successfully!', type: 'success' });
      } else {
        setMessage({ text: 'Failed to get an elaborated description. Please try again.', type: 'error' });
        console.error("Unexpected API response structure:", result);
      }
    } catch (error) {
      setMessage({ text: Error elaborating description: ${error.message}, type: 'error' });
      console.error("Error calling Gemini API:", error);
    } finally {
      setProjectLoadingStates(prev => ({ ...prev, [index]: false })); // Reset loading for specific project
    }
  };

  /**
   * Calls the Gemini API to generate an impact statement for an achievement.
   * @param {string} achievementTitle - The title of the achievement.
   * @param {string} achievementDescription - The description of the achievement.
   * @param {number} index - The index of the achievement in the array.
   */
  const generateAchievementImpact = async (achievementTitle, achievementDescription, index) => {
    setAchievementLoadingStates(prev => ({ ...prev, [index]: true })); // Set loading for specific achievement
    setMessage({ text: '', type: '' }); // Clear previous messages

    try {
      let chatHistory = [];
      const prompt = Generate a concise impact statement (around 30-50 words) for the following achievement. Focus on the key takeaway, the skill demonstrated, or the value added.\n\nAchievement Title: "${achievementTitle}"\nAchievement Description: "${achievementDescription}"\n\nImpact Statement:;
      chatHistory.push({ role: "user", parts: [{ text: prompt }] });

      const payload = { contents: chatHistory };
      const apiKey = ""; // Canvas will automatically provide the API key at runtime
      const apiUrl = https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey};

      const response = await fetch(apiUrl, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
      });

      if (!response.ok) {
        const errorData = await response.json();
        throw new Error(API error: ${response.status} - ${errorData.error?.message || response.statusText});
      }

      const result = await response.json();

      if (result.candidates && result.candidates.length > 0 &&
          result.candidates[0].content && result.candidates[0].content.parts &&
          result.candidates[0].content.parts.length > 0) {
        const text = result.candidates[0].content.parts[0].text;
        setAchievementImpacts(prev => ({ ...prev, [index]: text }));
        setMessage({ text: 'Achievement impact generated successfully!', type: 'success' });
      } else {
        setMessage({ text: 'Failed to generate achievement impact. Please try again.', type: 'error' });
        console.error("Unexpected API response structure:", result);
      }
    } catch (error) {
      setMessage({ text: Error generating impact: ${error.message}, type: 'error' });
      console.error("Error calling Gemini API for achievement impact:", error);
    } finally {
      setAchievementLoadingStates(prev => ({ ...prev, [index]: false })); // Reset loading for specific achievement
    }
  };

  return (
    <div className="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 font-inter text-gray-800 antialiased p-4 sm:p-6 lg:p-8">
      {/* Global CSS for animations */}
      <style>
        {`
        @keyframes fadeInUp {
          from {
            opacity: 0;
            transform: translateY(30px);
          }
          to {
            opacity: 1;
            transform: translateY(0);
          }
        }
        .animate-fade-in-up {
          animation: fadeInUp 0.8s ease-out forwards;
        }
        .message-box {
          position: fixed;
          top: 20px;
          left: 50%;
          transform: translateX(-50%);
          padding: 12px 24px;
          border-radius: 8px;
          box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
          z-index: 1000;
          display: flex;
          align-items: center;
          gap: 8px;
          font-weight: 500;
          opacity: 0;
          animation: fadeInOut 3s forwards;
        }
        .message-box.success {
          background-color: #d4edda;
          color: #155724;
          border: 1px solid #c3e6cb;
        }
        .message-box.error {
          background-color: #f8d7da;
          color: #721c24;
          border: 1px solid #f5c6cb;
        }
        @keyframes fadeInOut {
          0% { opacity: 0; transform: translateX(-50%) translateY(-20px); }
          10% { opacity: 1; transform: translateX(-50%) translateY(0); }
          90% { opacity: 1; transform: translateX(-50%) translateY(0); }
          100% { opacity: 0; transform: translateX(-50%) translateY(-20px); }
        }
        `}
      </style>

      {/* Message Box */}
      {message.text && (
        <div className={message-box ${message.type}}>
          {message.type === 'success' ? '✅' : '❌'} {message.text}
        </div>
      )}

      <div className="max-w-5xl mx-auto bg-white shadow-2xl rounded-3xl overflow-hidden border border-gray-200">
        {/* Hero Section */}
        <header className="relative bg-gradient-to-br from-indigo-700 to-purple-800 p-10 sm:p-16 text-white text-center rounded-t-3xl overflow-hidden">
          {/* Abstract background shapes */}
          <div className="absolute inset-0 z-0 opacity-20">
            <svg className="w-full h-full" viewBox="0 0 100 100" preserveAspectRatio="none">
              <circle cx="20" cy="20" r="15" fill="url(#grad1)"/>
              <circle cx="80" cy="80" r="20" fill="url(#grad2)"/>
              <path d="M0 50 Q 25 20, 50 50 T 100 50 L 100 0 L 0 0 Z" fill="url(#grad3)" opacity="0.5"/>
              <defs>
                <linearGradient id="grad1" x1="0%" y1="0%" x2="100%" y2="100%">
                  <stop offset="0%" stopColor="#818CF8" />
                  <stop offset="100%" stopColor="#C084FC" />
                </linearGradient>
                <linearGradient id="grad2" x1="0%" y1="0%" x2="100%" y2="100%">
                  <stop offset="0%" stopColor="#A78BFA" />
                  <stop offset="100%" stopColor="#6366F1" />
                </linearGradient>
                <linearGradient id="grad3" x1="0%" y1="0%" x2="100%" y2="100%">
                  <stop offset="0%" stopColor="#9CA3AF" />
                  <stop offset="100%" stopColor="#D1D5DB" />
                </linearGradient>
              </defs>
            </svg>
          </div>
          <div className="relative z-10">
            <h1 className="text-5xl sm:text-6xl font-extrabold mb-4 tracking-tight drop-shadow-lg animate-fade-in-up">
              {portfolio.name}
            </h1>
            <p className="text-xl sm:text-2xl font-light opacity-95 mb-8 leading-relaxed animate-fade-in-up" style={{ animationDelay: '0.2s' }}>
              {portfolio.title}
            </p>
            <div className="flex flex-wrap justify-center gap-x-8 gap-y-4 text-base sm:text-lg animate-fade-in-up" style={{ animationDelay: '0.4s' }}>
              <span className="flex items-center gap-2">
                <Globe size={20} className="text-purple-200" /> {portfolio.location}
              </span>
              <a href={mailto:${portfolio.email}} className="flex items-center gap-2 hover:underline hover:text-purple-200 transition-colors duration-300">
                <Mail size={20} className="text-purple-200" /> {portfolio.email}
              </a>
              <a href={portfolio.linkedin} target="_blank" rel="noopener noreferrer" className="flex items-center gap-2 hover:underline hover:text-purple-200 transition-colors duration-300">
                <Linkedin size={20} className="text-purple-200" /> LinkedIn
              </a>
              <a href={portfolio.github} target="_blank" rel="noopener noreferrer" className="flex items-center gap-2 hover:underline hover:text-purple-200 transition-colors duration-300">
                <Github size={20} className="text-purple-200" /> GitHub
              </a>
            </div>
          </div>
        </header>

        {/* Main Content Sections */}
        <main className="p-6 sm:p-10 space-y-16">
          {/* About Me Section */}
          <section
            id="about"
            ref={sectionRefs.about}
            className={bg-blue-50 p-8 rounded-2xl shadow-lg border border-blue-200 transform transition-all duration-700 ${getAnimationClasses('about')}}
          >
            <h2 className="text-3xl font-bold text-blue-800 mb-6 flex items-center gap-3">
              <Lightbulb size={28} className="text-blue-600" /> About Me
            </h2>
            <p className="text-gray-700 leading-relaxed text-lg">{portfolio.about}</p>
          </section>

          {/* Technical Skills Section */}
          <section
            id="skills"
            ref={sectionRefs.skills}
            className={bg-green-50 p-8 rounded-2xl shadow-lg border border-green-200 transform transition-all duration-700 ${getAnimationClasses('skills')}}
          >
            <h2 className="text-3xl font-bold text-green-800 mb-6 flex items-center gap-3">
              <Code size={28} className="text-green-600" /> Technical Skills
            </h2>
            <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-6">
              {Object.entries(portfolio.technicalSkills).map(([category, skills], catIndex) => (
                <div key={catIndex} className="bg-white p-5 rounded-xl shadow-md border border-green-100 transition-transform hover:scale-105 duration-300">
                  <h3 className="font-semibold text-xl text-green-900 mb-3 capitalize">{category}</h3>
                  <ul className="list-disc list-inside space-y-2 text-gray-700 text-base">
                    {skills.map((skill, index) => (
                      <li key={index}>{skill}</li>
                    ))}
                  </ul>
                </div>
              ))}
            </div>
          </section>

          {/* Projects Section */}
          <section
            id="projects"
            ref={sectionRefs.projects}
            className={bg-yellow-50 p-8 rounded-2xl shadow-lg border border-yellow-200 transform transition-all duration-700 ${getAnimationClasses('projects')}}
          >
            <h2 className="text-3xl font-bold text-yellow-800 mb-6 flex items-center gap-3">
              <Lightbulb size={28} className="text-yellow-600" /> Projects
            </h2>
            <div className="grid grid-cols-1 md:grid-cols-2 gap-8">
              {portfolio.projects.map((project, index) => (
                <div key={index} className="bg-white p-6 rounded-xl shadow-md border border-yellow-100 flex flex-col transition-transform hover:scale-105 duration-300">
                  <h3 className="font-semibold text-xl text-yellow-900 mb-3">{project.title}</h3>
                  <p className="text-gray-700 text-lg mb-4 flex-grow transition-all duration-500 ease-in-out">
                    {elaboratedDescriptions[index] || project.description}
                  </p>
                  <button
                    onClick={() => elaborateProjectDescription(project.title, project.description, index)}
                    className="mt-auto flex items-center justify-center gap-2 px-5 py-2 bg-yellow-600 text-white rounded-lg hover:bg-yellow-700 transition-colors duration-300 shadow-md disabled:opacity-50 disabled:cursor-not-allowed"
                    disabled={projectLoadingStates[index]}
                  >
                    {projectLoadingStates[index] ? (
                      <svg className="animate-spin h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                        <circle className="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="4"></circle>
                        <path className="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                      </svg>
                    ) : (
                      <Sparkles size={20} />
                    )}
                    {projectLoadingStates[index] ? 'Elaborating...' : '✨ Elaborate'}
                  </button>
                </div>
              ))}
            </div>
          </section>

          {/* Achievements Section */}
          <section
            id="achievements"
            ref={sectionRefs.achievements}
            className={bg-red-50 p-8 rounded-2xl shadow-lg border border-red-200 transform transition-all duration-700 ${getAnimationClasses('achievements')}}
          >
            <h2 className="text-3xl font-bold text-red-800 mb-6 flex items-center gap-3">
              <Award size={28} className="text-red-600" /> Achievements
            </h2>
            <div className="space-y-6">
              {portfolio.achievements.map((achievement, index) => (
                <div key={index} className="bg-white p-6 rounded-xl shadow-md border border-red-100 flex flex-col transition-transform hover:scale-105 duration-300">
                  <h3 className="font-semibold text-xl text-red-900 mb-2">{achievement.title}</h3>
                  <p className="text-gray-700 text-lg mb-4 flex-grow transition-all duration-500 ease-in-out">
                    {achievementImpacts[index] || achievement.description}
                  </p>
                  <button
                    onClick={() => generateAchievementImpact(achievement.title, achievement.description, index)}
                    className="mt-auto flex items-center justify-center gap-2 px-5 py-2 bg-red-600 text-white rounded-lg hover:bg-red-700 transition-colors duration-300 shadow-md disabled:opacity-50 disabled:cursor-not-allowed"
                    disabled={achievementLoadingStates[index]}
                  >
                    {achievementLoadingStates[index] ? (
                      <svg className="animate-spin h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                        <circle className="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="4"></circle>
                        <path className="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                      </svg>
                    ) : (
                      <Sparkles size={20} />
                    )}
                    {achievementLoadingStates[index] ? 'Generating Impact...' : '✨ Impact'}
                  </button>
                </div>
              ))}
            </div>
          </section>

          {/* Leadership & Teamwork Section */}
          <section
            id="leadership"
            ref={sectionRefs.leadership}
            className={bg-purple-50 p-8 rounded-2xl shadow-lg border border-purple-200 transform transition-all duration-700 ${getAnimationClasses('leadership')}}
          >
            <h2 className="text-3xl font-bold text-purple-800 mb-6 flex items-center gap-3">
              <Users size={28} className="text-purple-600" /> Leadership & Teamwork
            </h2>
            <ul className="list-disc list-inside space-y-3 text-gray-700 text-lg">
              {portfolio.leadershipTeamwork.map((item, index) => (
                <li key={index}>{item}</li>
              ))}
            </ul>
          </section>

          {/* Education Section */}
          <section
            id="education"
            ref={sectionRefs.education}
            className={bg-sky-50 p-8 rounded-2xl shadow-lg border border-sky-200 transform transition-all duration-700 ${getAnimationClasses('education')}}
          >
            <h2 className="text-3xl font-bold text-sky-800 mb-6 flex items-center gap-3">
              <GraduationCap size={28} className="text-sky-600" /> Education
            </h2>
            <div className="space-y-6">
              {portfolio.education.map((edu, index) => (
                <div key={index} className="bg-white p-6 rounded-xl shadow-md border border-sky-100 transition-transform hover:scale-105 duration-300">
                  <h3 className="font-semibold text-xl text-sky-900">{edu.degree}</h3>
                  <p className="text-gray-700">{edu.institution}</p>
                  <p className="text-base text-gray-600">{edu.details}</p>
                </div>
              ))}
            </div>
          </section>

          {/* Languages Section */}
          <section
            id="languages"
            ref={sectionRefs.languages}
            className={bg-teal-50 p-8 rounded-2xl shadow-lg border border-teal-200 transform transition-all duration-700 ${getAnimationClasses('languages')}}
          >
            <h2 className="text-3xl font-bold text-teal-800 mb-6 flex items-center gap-3">
              <MessageSquareText size={28} className="text-teal-600" /> Languages
            </h2>
            <ul className="list-disc list-inside space-y-3 text-gray-700 text-lg">
              {portfolio.languages.map((lang, index) => (
                <li key={index}>{lang.name}: {lang.proficiency}</li>
              ))}
            </ul>
          </section>
        </main>

        {/* Footer */}
        <footer className="bg-gray-900 text-white p-6 sm:p-8 text-center rounded-b-3xl">
          <p className="text-sm opacity-80">&copy; {new Date().getFullYear()} {portfolio.name}. All rights reserved.</p>
        </footer>
      </div>
    </div>
  );
}

export default App;
