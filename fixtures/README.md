# Demo and evaluation fixtures

아직 자료는 없고 폴더만 있다. 실존 환자·음성·문서를 넣지 않는다.

- seed: 환자+A(full)+B(companion)+C(schedule), 위임false, B첫동행과 가족 관찰 시드.
- audio: 준비한 가상음성; 실패 시 expected의 전사로 전환하고 fixture임을 표시.
- documents: 약20 모의 자료·흐린 사진·메모/약봉투 불일치.
- expected: 검증된 schedule/companion/full 결과, full진단·수치 가상값, 정답 라벨·혼입·재서술 샘플.

companion에 근거인용을 넣지 않는다. 검토본·공유본과 버전을 분리한다.
약 변경·질문을 실제 개인정보 없이 통일하고 정확도·누출률의 분모를 기록한다.
