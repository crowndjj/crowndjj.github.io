# crowndjj.github.io
/*
ArchitectPortfolio.tsx — TypeScript/TSX 호환 버전 (수정됨)

설명(요약)
- 원본 JSX가 "Unexpected token" 으로 빌드 타임에 에러가 난다는 리포트가 있어 코드
environment(파일 확장자 / tsconfig / babel 설정)와 JSX 파서를 모두 고려해 TypeScript(.
tsx) 환경에서도 바로 작동하도록 타입을 추가하고 안전한 문법으로 재작성했습니다.
- 이 파일을 그대로 `src/components/ArchitectPortfolio.tsx` 로 저장하면 TypeScript 프로젝트에
서 바로 사용하기 쉽습니다. (만약 JS-only 환경이면 `.jsx`로 저장해도 됩니다.)

주의: 에러가 계속되면 다음을 확인하세요:
  1) 파일 확장자가 `.tsx`(TypeScript 프로젝트)인지, `.jsx`(JS 프로젝트)인지 확인
  2) tsconfig.json의 "jsx" 설정이 적절한지 확인: "react-jsx"(React 17+ 권장) 또는 "react"
  3) Vite/CRA/Next 설정에서 babel 또는 swc가 JSX/TSX를 처리하도록 설정되어 있는지 확인

간단 사용법 (index.tsx / main.tsx 예시)
  // src/main.tsx (또는 src/index.tsx)
  import React from 'react';
  import { createRoot } from 'react-dom/client';
  import ArchitectPortfolio from './components/ArchitectPortfolio';
  import './index.css';

  const container = document.getElementById('root')!;
  const root = createRoot(container);
  root.render(
    <React.StrictMode>
      <ArchitectPortfolio />
    </React.StrictMode>
  );

TSConfig 권장 설정 (tsconfig.json 중 일부)
  {
    "compilerOptions": {
      "jsx": "react-jsx",
      "allowJs": true,
      "esModuleInterop": true,
      "skipLibCheck": true
    }
  }

문제 해결 힌트(Unexpected token 관련)
- **가장 흔한 원인:** 파일 확장자(.ts vs .tsx, .js vs .jsx)가 JSX를 포함한 파일과 맞지 않음
- **해결법:** 파일을 `.tsx`(TypeScript + JSX)로 저장하거나 `.jsx`로 저장하고 TypeScript가
  있는 프로젝트라면 `allowJs`를 켜고 `jsx` 옵션을 확인
- **추가 확인:** Vite 템플릿을 사용했다면 기본적으로 JSX 처리가 되어 있음. index 파일이
  `.ts`로 되어 있지 않은지 확인.

---

다음은 TypeScript로 타입을 명시한 안전한 컴포넌트 코드입니다.
(이 파일은 `src/components/ArchitectPortfolio.tsx`로 저장하세요.)
*/

import React, { useEffect, useRef, useState } from 'react';
import { motion } from 'framer-motion';
import { ChevronLeft, ChevronRight, X } from 'lucide-react';

interface Project {
  id: string;
  title: string;
  year: number;
  type: string;
  coverImage: string;
  images: string[];
  description: string;
  area: string;
  role: string;
  pdf: string | null;
  tags: string[];
}

const sampleProjects: Project[] = [
  {
    id: 'p1',
    title: '작업 A — 소형 주택 리노베이션',
    year: 2024,
    type: '주거',
    coverImage: '/images/project-a-cover.jpg',
    images: ['/images/project-a-1.jpg', '/images/project-a-2.jpg', '/images/project-a-3.jpg'],
    description:
      '콘셉트: 프라이버시를 지키는 방패형 메스. 평면 재구성과 빛의 흐름을 재배치하여 작은 부지에서 쾌적한 실내환경을 구현함.',
    area: '85 m²',
    role: '설계 · 시공감리',
    pdf: '/pdfs/project-a-report.pdf',
    tags: ['리노베이션', '주거', '프라이버시'],
  },
  {
    id: 'p2',
    title: '작업 B — 공공 전시장(소규모)',
    year: 2023,
    type: '공공',
    coverImage: '/images/project-b-cover.jpg',
    images: ['/images/project-b-1.jpg', '/images/project-b-2.jpg'],
    description:
      '전시 동선 최적화와 가변형 조명을 통해 다양한 전시 스케일에 대응하도록 설계. 재료는 저예산을 고려한 콘크리트·합판 조합.',
    area: '240 m²',
    role: '팀원(디자인 담당)',
    pdf: null,
    tags: ['전시', '공공', '콘크리트'],
  },
  {
    id: 'p3',
    title: '작업 C — 스터디 하우스',
    year: 2022,
    type: '상업',
    coverImage: '/images/project-c-cover.jpg',
    images: ['/images/project-c-1.jpg', '/images/project-c-2.jpg', '/images/project-c-3.jpg'],
    description:
      '커뮤니티 중심 소규모 상업공간. 내부 가구와 조명을 통해 다양한 사용성을 확보했고, 친환경 마감재 사용을 강조함.',
    area: '120 m²',
    role: '설계',
    pdf: '/pdfs/project-c-report.pdf',
    tags: ['상업', '커뮤니티', '지속가능'],
  },
];

export default function ArchitectPortfolio(): JSX.Element {
  const [projects] = useState<Project[]>(sampleProjects);
  const [query, setQuery] = useState<string>('');
  const [activeTag, setActiveTag] = useState<string>('All');
  const [selected, setSelected] = useState<Project | null>(null);
  const [slideIdx, setSlideIdx] = useState<number>(0);
  const modalRef = useRef<HTMLDivElement | null>(null);

  const tags: string[] = ['All', ...Array.from(new Set(projects.flatMap((p) => p.tags)))].slice(0, 20);

  useEffect(() => {
    function onKey(e: KeyboardEvent) {
      if (!selected) return;
      if (e.key === 'Escape') setSelected(null);
      if (e.key === 'ArrowRight') goSlide(1);
      if (e.key === 'ArrowLeft') goSlide(-1);
    }
    window.addEventListener('keydown', onKey);
    return () => window.removeEventListener('keydown', onKey);
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [selected, slideIdx]);

  function filtered(): Project[] {
    return projects.filter((p) => {
      const matchesTag = activeTag === 'All' || p.tags.includes(activeTag);
      const hay = [p.title, p.description, p.type, p.tags.join(' ')].join(' ').toLowerCase();
      const matchesQuery = hay.includes(query.toLowerCase());
      return matchesTag && matchesQuery;
    });
  }

  function openProject(p: Project) {
    setSelected(p);
    setSlideIdx(0);
    setTimeout(() => modalRef.current?.focus(), 0);
  }

  function closeProject() {
    setSelected(null);
  }

  function goSlide(dir: number) {
    if (!selected) return;
    const len = selected.images.length;
    setSlideIdx((s) => (s + dir + len) % len);
  }

  return (
    <div className="min-h-screen bg-slate-50 text-slate-800">
      {/* Header */}
      <header className="max-w-6xl mx-auto p-6 flex items-center justify-between">
        <div className="flex items-center gap-3">
          <div className="w-10 h-10 bg-slate-900 text-white rounded-2xl flex items-center justify-center font-semibold">SJ</div>
          <div>
            <h1 className="text-lg font-bold">성진 박 — Architecture Portfolio</h1>
            <p className="text-sm text-slate-500">건축 / 디자인 / 프로젝트 모음</p>
          </div>
        </div>
        <nav className="flex items-center gap-4">
          <a href="#projects" className="text-sm hover:underline">Projects</a>
          <a href="#about" className="text-sm hover:underline">About</a>
          <a href="#contact" className="text-sm hover:underline">Contact</a>
        </nav>
      </header>

      {/* Hero */}
      <section className="max-w-6xl mx-auto px-6 pb-8">
        <div className="rounded-2xl bg-white p-8 shadow-md grid gap-6 md:grid-cols-2 items-center">
          <div>
            <h2 className="text-3xl md:text-4xl font-extrabold leading-tight">작업을 개념부터 기록까지<br className="hidden md:inline" /> 한 곳에 모아두세요</h2>
            <p className="mt-3 text-slate-600">프로젝트 설명, 평면도, 모형사진, 시공사진, 보고서를 업로드해 전시형 포트폴리오를 만드세요. 아래에서 프로젝트를 찾아보세요.</p>
            <div className="mt-4 flex gap-3">
              <a href="#projects" className="inline-block rounded-xl px-4 py-2 bg-slate-900 text-white shadow">작업 보러가기</a>
              <a href="#contact" className="inline-block rounded-xl px-4 py-2 border border-slate-200">연락하기</a>
            </div>
          </div>
          <div className="order-first md:order-last">
            <div className="w-full h-48 md:h-56 rounded-2xl bg-gradient-to-tr from-slate-100 to-white shadow flex items-center justify-center">
              <div className="text-slate-400 text-center">대표 이미지 또는 모형 사진을 넣으세요</div>
            </div>
          </div>
        </div>
      </section>

      {/* Filters + Search */}
      <section id="projects" className="max-w-6xl mx-auto px-6">
        <div className="flex flex-col md:flex-row md:items-center md:justify-between gap-4 mb-4">
          <div className="flex gap-2 flex-wrap">
            {tags.map((t) => (
              <button
                key={t}
                onClick={() => setActiveTag(t)}
                className={`text-sm px-3 py-1 rounded-full border ${activeTag === t ? 'bg-slate-900 text-white' : 'bg-white text-slate-700'}`}
              >
                {t}
              </button>
            ))}
          </div>
          <div className="flex items-center gap-2">
            <input
              value={query}
              onChange={(e) => setQuery(e.target.value)}
              placeholder="프로젝트 검색 (제목, 설명, 태그...)"
              className="px-3 py-2 rounded-xl border w-56 focus:outline-none"
              aria-label="프로젝트 검색"
            />
            <div className="text-sm text-slate-500">{filtered().length} 개</div>
          </div>
        </div>

        {/* Projects grid */}
        <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6">
          {filtered().map((p) => (
            <motion.article
              key={p.id}
              initial={{ opacity: 0, y: 8 }}
              animate={{ opacity: 1, y: 0 }}
              transition={{ duration: 0.3 }}
              className="bg-white rounded-2xl shadow p-3"
            >
              <div className="relative rounded-xl overflow-hidden">
                <img src={p.coverImage} alt={`${p.title} 대표 이미지`} className="w-full h-44 object-cover block" />
                <div className="absolute inset-0 bg-gradient-to-t from-black/30 flex items-end p-3">
                  <div>
                    <h3 className="text-white text-lg font-semibold">{p.title}</h3>
                    <p className="text-xs text-white/80">{p.year} · {p.type} · {p.area}</p>
                  </div>
                </div>
              </div>
              <div className="mt-3 flex items-center justify-between">
                <div className="text-sm text-slate-600 line-clamp-2">{p.description}</div>
                <div className="flex items-center gap-2">
                  <button onClick={() => openProject(p)} className="px-3 py-1 rounded-xl border">열기</button>
                </div>
              </div>
            </motion.article>
          ))}
        </div>
      </section>

      {/* About */}
      <section id="about" className="max-w-6xl mx-auto px-6 mt-10">
        <div className="bg-white rounded-2xl shadow p-6">
          <h3 className="text-xl font-bold">About / 소개</h3>
          <p className="mt-2 text-slate-600">안녕하세요. 저는 건축을 전공하는 학생(또는 실무자)입니다. 학교 작업과 소규모 실무 프로젝트를 중심으로 설계 과정을 정리해두었습니다. 각 프로젝트에는 콘셉트, 평면도, 단면도, 모형사진, 시공사진, 최종보고서(PDF)를 연결해 두세요.</p>
          <ul className="mt-3 grid sm:grid-cols-2 gap-3 text-sm">
            <li>• 프로젝트별 핵심: 개념 / 공간 구성 / 재료 / 조명</li>
            <li>• 이미지 권장: 너비 1600px 이상, JPG/WEBP로 압축</li>
            <li>• 파일명 규칙: YYYY-프로젝트명-번호.jpg</li>
            <li>• 추천 레이아웃: 표지(썸네일) → 설명 → 이미지 순</li>
          </ul>
        </div>
      </section>

      {/* Contact */}
      <section id="contact" className="max-w-6xl mx-auto px-6 mt-8 mb-16">
        <div className="bg-white rounded-2xl shadow p-6 grid md:grid-cols-2 gap-6 items-center">
          <div>
            <h4 className="text-lg font-bold">Contact</h4>
            <p className="text-slate-600 mt-2">포트폴리오에 관해 수정하거나, 협업 문의가 있다면 아래로 연락주세요.</p>
            <div className="mt-4">
              <a href="mailto:your.email@example.com" className="inline-block rounded-xl px-4 py-2 border">Email 보내기</a>
            </div>
          </div>
          <form className="space-y-3">
            <input className="w-full px-3 py-2 rounded-xl border" placeholder="이름" />
            <input className="w-full px-3 py-2 rounded-xl border" placeholder="이메일" />
            <textarea className="w-full px-3 py-2 rounded-xl border" placeholder="메시지" rows={4} />
            <button type="button" className="px-4 py-2 rounded-xl bg-slate-900 text-white">보내기</button>
          </form>
        </div>
      </section>

      {/* Footer */}
      <footer className="max-w-6xl mx-auto px-6 pb-12 text-sm text-slate-500">
        <div className="pt-6 border-t border-slate-100">© {new Date().getFullYear()} 성진 박 — All rights reserved.</div>
      </footer>

      {/* Modal: 선택한 프로젝트 상세 */}
      {selected && (
        <div
          role="dialog"
          aria-modal="true"
          className="fixed inset-0 z-50 flex items-center justify-center p-6"
        >
          <div className="absolute inset-0 bg-black/50" onClick={closeProject} />

          <motion.div
            ref={modalRef}
            tabIndex={-1}
            initial={{ opacity: 0, scale: 0.98 }}
            animate={{ opacity: 1, scale: 1 }}
            transition={{ duration: 0.18 }}
            className="relative z-10 w-full max-w-4xl bg-white rounded-2xl shadow-lg overflow-hidden"
          >
            <div className="flex items-start justify-between p-4 border-b">
              <div>
                <h3 className="text-lg font-bold">{selected.title}</h3>
                <div className="text-sm text-slate-500">{selected.year} · {selected.type} · {selected.area} · {selected.role}</div>
              </div>
              <button onClick={closeProject} aria-label="닫기" className="p-2 rounded-md hover:bg-slate-100">
                <X size={18} />
              </button>
            </div>

            <div className="p-4 grid gap-4 md:grid-cols-3">
              <div className="md:col-span-2">
                <div className="relative">
                  <img src={selected.images[slideIdx]} alt={`${selected.title} 이미지 ${slideIdx + 1}`} className="w-full h-72 object-cover rounded-xl" />

                  <button onClick={() => goSlide(-1)} className="absolute left-3 top-1/2 -translate-y-1/2 bg-white/80 rounded-full p-2 shadow">
                    <ChevronLeft />
                  </button>
                  <button onClick={() => goSlide(1)} className="absolute right-3 top-1/2 -translate-y-1/2 bg-white/80 rounded-full p-2 shadow">
                    <ChevronRight />
                  </button>
                </div>

                {/* 썸네일 라인 */}
                <div className="mt-3 flex gap-2 overflow-auto">
                  {selected.images.map((img, i) => (
                    <button key={img} onClick={() => setSlideIdx(i)} className={`rounded-lg overflow-hidden ${i === slideIdx ? 'ring-2 ring-slate-900' : ''}`}>
                      <img src={img} alt={`썸네일 ${i + 1}`} className="w-20 h-14 object-cover block" />
                    </button>
                  ))}
                </div>
              </div>

              <div>
                <div className="text-slate-700 text-sm">{selected.description}</div>
                <div className="mt-3">
                  <div className="text-xs text-slate-500">Tags</div>
                  <div className="mt-2 flex flex-wrap gap-2">
                    {selected.tags.map((t) => (
                      <span key={t} className="text-xs px-2 py-1 rounded-full border">{t}</span>
                    ))}
                  </div>
                </div>

                <div className="mt-4 flex gap-2">
                  {selected.pdf ? (
                    <a href={selected.pdf} className="inline-block px-3 py-2 rounded-xl border" download>
                      보고서(PDF) 다운로드
                    </a>
                  ) : (
                    <button className="px-3 py-2 rounded-xl border" disabled>
                      보고서 없음
                    </button>
                  )}

                  <a href={`mailto:your.email@example.com?subject=Portfolio: ${encodeURIComponent(selected.title)}`} className="px-3 py-2 rounded-xl bg-slate-900 text-white">문의하기</a>
                </div>

                <div className="mt-4 text-xs text-slate-400">권장 이미지 규격: 1600px 이상 너비, webp/jpg로 압축. 파일명 규칙: YYYY-프로젝트-숫자.jpg</div>
              </div>
            </div>
          </motion.div>
        </div>
      )}
    </div>
  );
}

