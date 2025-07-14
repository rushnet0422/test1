document.addEventListener('DOMContentLoaded', () => {
    const noteInput = document.getElementById('note-input');
    const addNoteBtn = document.getElementById('add-note-btn');
    const notesContainer = document.getElementById('notes-container');

    // 메모 로드 및 표시
    function loadNotes() {
        notesContainer.innerHTML = ''; // 기존 메모 삭제
        let notes = getNotes();
        notes.forEach((note, index) => {
            addNoteToDOM(note, index);
        });
    }

    // 로컬 스토리지에서 메모 가져오기
    function getNotes() {
        return JSON.parse(localStorage.getItem('notes')) || [];
    }

    // 로컬 스토리지에 메모 저장하기
    function saveNotes(notes) {
        localStorage.setItem('notes', JSON.stringify(notes));
    }

    // 메모를 DOM에 추가 (HTML 요소 생성)
    function addNoteToDOM(noteText, index) {
        const noteItem = document.createElement('div');
        noteItem.classList.add('note-item');
        noteItem.dataset.index = index; // 인덱스 저장

        const noteContent = document.createElement('span');
        noteContent.classList.add('note-content');
        noteContent.textContent = noteText;

        const noteActions = document.createElement('div');
        noteActions.classList.add('note-actions');

        const editBtn = document.createElement('button');
        editBtn.classList.add('edit-btn');
        editBtn.innerHTML = '✏️'; // 연필 이모지
        editBtn.title = '수정';

        const deleteBtn = document.createElement('button');
        deleteBtn.classList.add('delete-btn');
        deleteBtn.innerHTML = '🗑️'; // 휴지통 이모지
        deleteBtn.title = '삭제';

        noteActions.appendChild(editBtn);
        noteActions.appendChild(deleteBtn);

        noteItem.appendChild(noteContent);
        noteItem.appendChild(noteActions);
        notesContainer.prepend(noteItem); // 최신 메모를 상단에 추가

        // 삭제 버튼 이벤트 리스너
        deleteBtn.addEventListener('click', () => {
            deleteNote(index);
        });

        // 수정 버튼 이벤트 리스너
        editBtn.addEventListener('click', () => {
            editNote(noteItem, noteContent, index);
        });
    }

    // 새 메모 추가
    addNoteBtn.addEventListener('click', () => {
        const newNote = noteInput.value.trim();
        if (newNote) {
            let notes = getNotes();
            notes.push(newNote);
            saveNotes(notes);
            noteInput.value = ''; // 입력 필드 초기화
            loadNotes(); // 메모 목록 새로고침
        } else {
            alert('메모 내용을 입력해주세요!');
        }
    });

    // 메모 삭제
    function deleteNote(indexToDelete) {
        let notes = getNotes();
        // filter를 사용하여 삭제할 인덱스를 제외한 새로운 배열 생성
        notes = notes.filter((_, index) => index !== indexToDelete);
        saveNotes(notes);
        loadNotes(); // 메모 목록 새로고침
    }

    // 메모 수정
    function editNote(noteItem, noteContentSpan, indexToEdit) {
        const currentText = noteContentSpan.textContent;
        const inputField = document.createElement('textarea');
        inputField.value = currentText;
        inputField.classList.add('edit-textarea');
        inputField.style.width = '100%';
        inputField.style.height = 'auto'; // 높이 자동 조절
        inputField.style.minHeight = '60px'; // 최소 높이

        // 기존 텍스트와 액션 버튼 숨기기
        noteContentSpan.style.display = 'none';
        noteItem.querySelector('.note-actions').style.display = 'none';

        // 입력 필드를 메모 아이템에 추가
        noteItem.insertBefore(inputField, noteItem.firstChild);
        inputField.focus();

        // 저장 버튼과 취소 버튼 추가
        const saveEditBtn = document.createElement('button');
        saveEditBtn.textContent = '저장';
        saveEditBtn.classList.add('save-edit-btn');

        const cancelEditBtn = document.createElement('button');
        cancelEditBtn.textContent = '취소';
        cancelEditBtn.classList.add('cancel-edit-btn');

        const editActionsDiv = document.createElement('div');
        editActionsDiv.classList.add('edit-actions-temp');
        editActionsDiv.appendChild(saveEditBtn);
        editActionsDiv.appendChild(cancelEditBtn);
        noteItem.appendChild(editActionsDiv);

        saveEditBtn.addEventListener('click', () => {
            const updatedText = inputField.value.trim();
            if (updatedText) {
                let notes = getNotes();
                notes[indexToEdit] = updatedText;
                saveNotes(notes);
                loadNotes(); // 메모 목록 새로고침
            } else {
                alert('메모 내용은 비워둘 수 없습니다!');
                inputField.focus();
            }
        });

        cancelEditBtn.addEventListener('click', () => {
            // 수정 모드 취소 시 원래 상태로 복구
            noteItem.removeChild(inputField);
            noteItem.removeChild(editActionsDiv);
            noteContentSpan.style.display = 'block';
            noteItem.querySelector('.note-actions').style.display = 'flex';
        });

        // 엔터 키로 저장, Esc 키로 취소
        inputField.addEventListener('keydown', (e) => {
            if (e.key === 'Enter' && !e.shiftKey) { // Shift+Enter는 줄바꿈
                e.preventDefault();
                saveEditBtn.click();
            } else if (e.key === 'Escape') {
                cancelEditBtn.click();
            }
        });
    }

    // 초기 로드
    loadNotes();
});