
### 공람자 추가 - 이미 존재하는 열람자인지 확인
```
// 결재 공유 열람자 목록을 확인하여 이미 존재하는 열람자인지 확인
for (ApprovalShareReaderDto readerDto : createCommandInDto.getApprovalShareReaderList()) {
    // 데이터베이스에서 해당 결재 공유와 사용 중인 상태인 열람자를 조회합니다.
    Optional<ApprovalShareReader> existingReader = approvalShareReaderRepository
            .findByApprovalShareAndUserIdAndUseYn(approvalShare, readerDto.getUserId(), YesNo.Y);

    if (existingReader.isPresent()) {
        // 이미 존재하는 열람자라면 예외를 던지거나, 상황에 따라 처리합니다.
        throw new DuplicateReaderException("사용자 ID " + readerDto.getUserId() + "로 이미 존재하는 결재 공유 열람자가 있습니다");
    }
}

// 기존 로직: 결재 공유 열람자 화면 생성
List<ApprovalShareReaderView> approvalShareReaderViewList = createApprovalShareReaderViewList(createCommandInDto.getApprovalShareReaderList());
List<String> approvalShareReaderViewIdList = approvalShareReaderViewList.stream().map(ApprovalShareReaderView::getApprovalShareReaderViewId).collect(Collectors.toList());

// 기존 로직: 결재 공유 공유자 화면 생성
ApprovalShareSharerView approvalShareSharerView = createApprovalShareSharerView(createCommandInDto.getApprovalShareSharer());

// 기존 로직: 결재 공유 생성
ApprovalShare approvalShare = createApprovalShare(createCommandInDto.getParentApprovalLineId(), approvalShareReaderViewList, approvalShareSharerView, approvalLine);

// 결과 반환
return ApprovalShareCreateCommandOutDto.builder()
        .approvalShareId(approvalShare.getApprovalShareId())
        .approvalShareSharerViewId(approvalShareSharerView.getApprovalShareSharerViewId())
        .approvalShareReaderViewIdList(approvalShareReaderViewIdList)
        .build();

```
---> approvalShareReaderRepository.findByApprovalShareAndUserIdAndUseYn는 데이터베이스에서 결재 공유와 사용 중인 열람자를 조회하는 가상의 메서드입니다. 

### 결재선 존재하는지 확인하는 쿼리
```
@EntityGraph(attributePaths = {
            "integratedApprovalLineInterfaceParameter",
            "receive",
            "sendList"
    })
    Optional<ApprovalLine> findOneToOneFetchByApprovalLineIdAndUseYn(String approvalLineId, YesNo yesNo);
```
해석해줘

이 코드는 Spring Data JPA에서 사용되는 @EntityGraph 어노테이션을 이용한 쿼리 메서드입니다. 이 쿼리 메서드는 ApprovalLine 엔터티를 조회할 때 특정 연관 관계들을 함께 가져오도록 정의되어 있습니다.

@EntityGraph: 이 어노테이션은 엔터티 그래프를 사용하여 연관된 엔터티들을 로딩할 때 발생할 수 있는 N+1 문제를 해결하는 데 사용됩니다. 즉, 특정 연관 관계를 함께 로딩하여 성능을 최적화합니다.

attributePaths: 연관된 엔터티들을 함께 로딩할 때 어떤 속성들을 가져올지를 지정합니다. 이 코드에서는 integratedApprovalLineInterfaceParameter, receive, sendList 속성들을 함께 가져오도록 지정되어 있습니다.

Optional<ApprovalLine>: 이 메서드는 ApprovalLine을 반환하며, 결과가 없을 경우 Optional.empty()를 반환합니다.

findOneToOneFetchByApprovalLineIdAndUseYn(String approvalLineId, YesNo yesNo): 이 메서드는 approvalLineId와 yesNo를 파라미터로 받아 결재선을 조회하는 쿼리 메서드입니다. yesNo는 Yes 또는 No 값을 가지는 열거형(여기서는 YesNo)이며, 사용 여부를 나타냅니다.

따라서, 이 메서드를 호출하면 ApprovalLine 엔터티와 연관된 integratedApprovalLineInterfaceParameter, receive, sendList 속성들이 함께 로딩되어 반환됩니다.
