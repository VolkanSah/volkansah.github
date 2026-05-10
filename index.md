---
layout: default
title: My GitHub Repositories
---

<style>
/* Repo card shimmer on load */
@keyframes card-in {
    from { opacity: 0; transform: translateY(16px); }
    to   { opacity: 1; transform: translateY(0); }
}
.repo-card-wrap { animation: card-in 0.4s ease both; }

/* Stagger via JS-injected --i */
.repo-card-wrap { animation-delay: calc(var(--i, 0) * 0.05s); }

/* Card label tag */
.repo-tag {
    display: inline-block;
    font-family: 'Space Mono', monospace;
    font-size: 10px;
    padding: 2px 8px;
    border-radius: 4px;
    background: rgba(88,166,255,0.10);
    border: 1px solid rgba(88,166,255,0.25);
    color: #58A6FF;
    margin-bottom: 8px;
    letter-spacing: 0.06em;
}

/* Stars badge */
.star-badge {
    font-family: 'Space Mono', monospace;
    font-size: 11px;
    color: #8B949E;
    float: right;
}
.star-badge i { color: rgba(239,186,0,0.7); margin-right: 3px; }

/* View Details button glow pulse */
.btn-view {
    position: relative;
    overflow: hidden;
}
.btn-view::after {
    content: '';
    position: absolute;
    inset: 0;
    border-radius: 6px;
    background: radial-gradient(ellipse at 50% 120%, rgba(88,166,255,0.15), transparent 70%);
    opacity: 0;
    transition: opacity 0.3s;
}
.btn-view:hover::after { opacity: 1; }

/* Modal body scrollbar thin */
.modal-body { max-height: 65vh; overflow-y: auto; }

/* README images in modal */
.modal-body img {
    max-width: 100%;
    border-radius: 6px;
    border: 1px solid rgba(255,255,255,0.08);
}

/* README headings */
.modal-body h1, .modal-body h2, .modal-body h3,
.modal-body h4, .modal-body h5 {
    color: #E6EDF3;
    font-family: 'Syne', sans-serif;
}

/* README code blocks */
.modal-body code {
    background: rgba(33,38,45,0.8);
    border: 1px solid rgba(255,255,255,0.07);
    border-radius: 4px;
    padding: 1px 5px;
    font-family: 'Space Mono', monospace;
    font-size: 12px;
    color: #3DDBD9;
}
.modal-body pre code {
    background: transparent;
    border: none;
    padding: 0;
}
.modal-body pre {
    background: rgba(33,38,45,0.75);
    border: 1px solid rgba(255,255,255,0.07);
    border-radius: 8px;
    padding: 14px;
    overflow-x: auto;
    font-size: 13px;
    line-height: 1.6;
}

/* Table in README */
.modal-body table {
    width: 100%;
    border-collapse: collapse;
    font-size: 13px;
    font-family: 'Space Mono', monospace;
}
.modal-body table th, .modal-body table td {
    padding: 8px 12px;
    border: 1px solid rgba(255,255,255,0.08);
}
.modal-body table th { background: rgba(88,166,255,0.08); color: #58A6FF; }

/* Pagination */
.pagination-wrap {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 12px;
    margin-top: 40px;
    font-family: 'Space Mono', monospace;
    font-size: 12px;
    color: #8B949E;
}
#pageIndicator {
    padding: 4px 14px;
    border-radius: 6px;
    background: rgba(255,255,255,0.04);
    border: 1px solid rgba(255,255,255,0.08);
}

/* Loading shimmer */
.repo-skeleton {
    background: linear-gradient(90deg,
        rgba(255,255,255,0.03) 25%,
        rgba(255,255,255,0.07) 50%,
        rgba(255,255,255,0.03) 75%);
    background-size: 200% 100%;
    animation: shimmer 1.5s infinite;
    border-radius: 12px;
    min-height: 140px;
}
@keyframes shimmer {
    0%   { background-position: 200% 0; }
    100% { background-position: -200% 0; }
}
</style>

<div class="container">
    <div class="row" id="repo-list" data-masonry='{"percentPosition": true}'></div>
    <div class="pagination-wrap">
        <button id="prevPage" class="btn btn-secondary" onclick="loadPrevPage()">← prev</button>
        <span id="pageIndicator">page 1</span>
        <button id="nextPage" class="btn btn-secondary" onclick="loadNextPage()">next →</button>
    </div>
</div>

<script>
let currentPage = 1;
const perPage = 18;

function fetchAllRepos(page = 1) {
    const repoList = document.getElementById('repo-list');

    // Show skeletons while loading
    repoList.innerHTML = Array.from({length: 6}, () =>
        `<div class="col-md-4 mb-4"><div class="repo-skeleton"></div></div>`
    ).join('');

    fetch(`https://api.github.com/users/volkansah/repos?type=owner&sort=updated&per_page=${perPage}&page=${page}`)
        .then(response => {
            updatePaginationButtons(response.headers.get('Link'));
            return response.json();
        })
        .then(data => {
            repoList.innerHTML = '';
            document.getElementById('pageIndicator').textContent = `page ${page}`;

            const filtered = data.filter(repo =>
                !repo.fork &&
                repo.name !== 'volkansah.github.io' &&
                repo.name !== 'VolkanSah'
            );

            filtered.forEach((repo, index) => {
                const wrap = document.createElement('div');
                wrap.className = 'col-md-4 repo-card-wrap';
                wrap.style.setProperty('--i', index);

                const stars = repo.stargazers_count
                    ? `<span class="star-badge"><i class="fas fa-star"></i>${repo.stargazers_count}</span>`
                    : '';
                const lang = repo.language
                    ? `<span class="repo-tag">${repo.language}</span>`
                    : '';

                wrap.innerHTML = `
                    <div class="card mb-4">
                        <div class="card-body">
                            ${stars}
                            ${lang}
                            <h5 class="card-title">${repo.name}</h5>
                            <p class="card-text">${repo.description || '// no description'}</p>
                            <button class="btn btn-primary btn-view"
                                data-toggle="modal"
                                data-target="#repoModal-${index}"
                                onclick="loadReadme('${repo.full_name}', ${index})">
                                View Details
                            </button>
                        </div>
                    </div>

                    <div class="modal fade" id="repoModal-${index}" tabindex="-1" role="dialog" aria-labelledby="repoModalLabel-${index}" aria-hidden="true">
                        <div class="modal-dialog modal-lg" role="document">
                            <div class="modal-content">
                                <div class="modal-header">
                                    <h3 class="modal-title" id="repoModalLabel-${index}">${repo.name}</h3>
                                    <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                                        <span aria-hidden="true">&times;</span>
                                    </button>
                                </div>
                                <div class="modal-body" id="repoContent-${index}">
                                    <p style="color:#8B949E;font-family:'Space Mono',monospace;font-size:12px;">// loading readme...</p>
                                </div>
                                <div class="modal-footer">
                                    <a href="${repo.html_url}" target="_blank" class="btn btn-primary">
                                        <i class="fab fa-github" style="margin-right:6px;"></i>Go to Repository
                                    </a>
                                    <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                                </div>
                            </div>
                        </div>
                    </div>
                `;
                repoList.appendChild(wrap);
            });

            imagesLoaded(repoList, () => {
                new Masonry(repoList, {
                    itemSelector: '.col-md-4',
                    percentPosition: true
                });
            });
        })
        .catch(() => {
            repoList.innerHTML = '<p style="color:#8B949E;font-family:\'Space Mono\',monospace;padding:2rem;">// error loading repositories.</p>';
        });
}

function updatePaginationButtons(linkHeader) {
    const links = parseLinkHeader(linkHeader);
    document.getElementById('prevPage').disabled = !links.prev;
    document.getElementById('nextPage').disabled = !links.next;
}

function parseLinkHeader(header) {
    if (!header) return {};
    return Object.fromEntries(
        header.split(',').map(p => {
            const [url, rel] = p.split(';');
            return [rel.replace(/rel="(.*)"/, '$1').trim(), url.replace(/<(.*)>/, '$1').trim()];
        })
    );
}

function loadPrevPage() {
    if (currentPage > 1) { currentPage--; fetchAllRepos(currentPage); }
}
function loadNextPage() {
    currentPage++;
    fetchAllRepos(currentPage);
}

function loadReadme(repoFullName, index) {
    fetch(`https://api.github.com/repos/${repoFullName}/readme`, {
        headers: { 'Accept': 'application/vnd.github.v3.html' }
    })
    .then(r => r.text())
    .then(data => {
        const repoUrl = `https://github.com/${repoFullName}/blob/master/`;
        data = data.replace(/href="#([^"]+)"/g, `href="#repoContent-${index}-$1"`);
        data = data.replace(/id="([^"]+)"/g, `id="repoContent-${index}-$1"`);
        data = data.replace(/<h([1-6])([^>]*)id="([^"]+)"([^>]*)>/g, `<h$1$2id="repoContent-${index}-$3"$4>`);
        data = data.replace(/src="([^"]+)"/g, (match, p1) => {
            if (!p1.startsWith('http') && !p1.startsWith('//')) return `src="${repoUrl}${p1}"`;
            return match;
        });
        const el = document.getElementById(`repoContent-${index}`);
        el.innerHTML = data;
        if (window.MathJax && MathJax.typesetPromise) {
            MathJax.typesetPromise([el]).catch(e => console.error('MathJax:', e));
        } else if (window.renderMathInElement) {
            renderMathInElement(el, {
                delimiters: [
                    {left:"$$",right:"$$",display:true},
                    {left:"$",right:"$",display:false},
                    {left:"\\(",right:"\\)",display:false},
                    {left:"\\[",right:"\\]",display:true}
                ]
            });
        }
    })
    .catch(() => {
        document.getElementById(`repoContent-${index}`).innerHTML = '<p>// README could not be loaded.</p>';
    });
}

fetchAllRepos();
</script>
