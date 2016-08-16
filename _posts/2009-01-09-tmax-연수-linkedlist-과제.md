---
layout: post
title: "[Tmax 연수] LinkedList 과제"
date: 2009-01-09 21:21:01.000000000 +09:00
type: post
published: true
status: publish
categories:
- Programming
tags:
- Tmax 연수
- C
---

```c
#include <stdio.h>
#include <stdlib.h>
#include <memory.h>
#include <assert.h>

// doubleLinkedList.h
// ————————————————-
typedef enum { FALSE,
    TRUE } BOOL;
typedef int Data;
//typedef struct _Data { // 데이터 영역 확장 고려
// int age;
//} Data;
typedef struct _Node Node;
typedef struct _Node {
    Node* next; // PREVIOUS Node
    Node* previous; // NEXT Node
} Node;

typedef struct _linkedList {
    Node* head;
    Node* tail;
    int length;
    Node* current;
} LinkedList;
// ————————————————-
Node* CreateEmptyNode(size_t size);
void Create(LinkedList* linkedList);
Node* AppendFromHead(LinkedList* linkedList, void* item, size_t size);
Node* AppendFromTail(LinkedList* linkedList, void* item, size_t size);
Node* InsertAfter(LinkedList* linkedList, Node* index, void* item, size_t size);
Node* InsertBefore(LinkedList* linkedList, Node* index, void* item, size_t size);
Node* Delete(LinkedList* linkedList, Node* index, void (*dataFree)(void*));
void GetAt(LinkedList* linkedList, Node* index, void* getData, size_t size);
void Destroy(LinkedList* linkedList, void (*dataFree)(void*));
Node* LinearSearchUnique(LinkedList* linkedList, void* key, int (*dataCompare)(void*, void*));
void LinearSearchDuplicate(LinkedList* linkedList, Node*** indexes, int* count, void* key, int (*dataCompare)(void*, void*));
void Sort(LinkedList* linkedList, size_t size, int (*dataCompare)(void*, void*));
void Display(LinkedList* linkedList, void (*dataPrint)(void*));
// ————————————————-
//————————————————————————————————————
//  값 출력 함수
//————————————————————————————————————
void nodeDataPrint(void* p)
{
    printf("%5d", *((Data*)p));
}
//————————————————————————————————————
//  데이터 비교 함수
//————————————————————————————————————
int integerCompare(void* p1, void* p2)
{
    // 구조체 사용시 구조체 내부의 비교 기준
    //
    //if( ((Data*)p1)->age > ((Data*)p2)->age )
    // return 1;
    //else if( ((Data*)p1)->age < ((Data*)p2)->age )
    // return -1;
    //else
    // return 0;
    if (*((Data*)p1) > *((Data*)p2))
        return 1;
    else if (*((Data*)p1) < *((Data*)p2))
        return -1;
    else
        return 0;
}
//————————————————————————————————————
// Empty 노드 생성하고 주소를 리턴
// isCreateData : 데이터 영역 동적 할당 여부(1:할당,0:취소)
//————————————————————————————————————
Node* CreateEmptyNode(size_t size)
{
    Node* tmp = (Node*)malloc(sizeof(Node) + size);
    assert(tmp != NULL); // 메모리 할당이 실패하면 종료

    // 노드 초기화
    tmp->previous = NULL;
    tmp->next = NULL;
    // 노드 시작 주소값 리턴
    return (tmp != NULL) ? tmp : NULL;
}
//————————————————————————————————————
//  head노드 다음에 데이터 추가
//————————————————————————————————————
Node* AppendFromHead(LinkedList* linkedList, void* item, size_t size)
{
    linkedList->current = CreateEmptyNode(size); // 새 노드의 동적 메모리 할당
    assert(linkedList->current != NULL); // 메모리 할당이 실패하면 종료
    memcpy(linkedList->current + 1, item, size); // 추가할 데이터를 새 노드의 데이터 영역에 저장
    // head노드 뒤에 새 노드 연결
    linkedList->current->previous = linkedList->head; // 새 노드의 previous로 head노드를 가리키게 한다
    linkedList->current->next = linkedList->head->next; // 새 노드의 next로 head노드의 다음 노드를 가리키게 한다
    linkedList->head->next->previous = linkedList->current; // 새 노드 다음 노드가 새 노드를 가리키게 한다
    linkedList->head->next = linkedList->current; // 새 노드 이전 노드가 새 노드를 가리키게 한다

    linkedList->length++; // 노드의 갯수를 하나 증가 시킨다
    return linkedList->current; // 새 노드의 포인터를 리턴한다
}

//————————————————————————————————————
//  tail노드 이전에 데이터 추가
//————————————————————————————————————
Node* AppendFromTail(LinkedList* linkedList, void* item, size_t size)
{
    assert(linkedList->head != NULL);
    linkedList->current = CreateEmptyNode(size); // 새 노드의 동적 메모리 할당
    assert(linkedList->current != NULL); // 메모리 할당이 실패하면 종료
    memcpy(linkedList->current + 1, item, size); // 추가할 데이터를 새 노드의 데이터 영역에 저장
    linkedList->current->next = linkedList->current;
    linkedList->current->previous = linkedList->tail;
    // tail노드 앞에 새 노드 연결
    linkedList->current->next = linkedList->tail; // 새 노드의 next로 tail노드를 가리키게 한다
    linkedList->current->previous = linkedList->tail->previous; // 새 노드의 previous로 tail노드의 이전 노드를 가리키게 한다
    linkedList->tail->previous->next = linkedList->current; // 새 노드 이전 노드가 새 노드를 가리키게 한다
    linkedList->tail->previous = linkedList->current; // 새 노드 다음 노드가 새 노드를 가리키게 한다
    linkedList->length++; // 노드의 갯수를 하나 증가 시킨다
    return linkedList->current; // 새 노드의 포인터를 리턴한다
}

//————————————————————————————————————
//  링크드리스트의 모든 데이터 출력
//————————————————————————————————————
void Display(LinkedList* linkedList, void (*dataPrint)(void*))
{
    int i;
    linkedList->current = linkedList->head->next; // current포인터로 처음 데이터노드를 가리키게 한다
    for (i = 0; i < linkedList->length; i++) { // 데이터의 수 만큼 반복한다
        dataPrint((Node*)(linkedList->current + 1)); // 전달 받은 함수로 데이터 부분을 출력한다
        linkedList->current = linkedList->current->next; // current포인터를 다음 노드로 이동한다
    }
}
//————————————————————————————————————
//  링크드리스트의 특정 인덱스의 데이터를 가져옴
//————————————————————————————————————
void GetAt(LinkedList* linkedList, Node* index, void* getData, size_t size)
{
    memcpy(getData, (Node*)(index + 1), size);
}

//————————————————————————————————————
//  링크드리스트의 모든 데이터 오름차순 정렬
//————————————————————————————————————
void Sort(LinkedList* linkedList, size_t size, int (*dataCompare)(void*, void*))
{
    Node *i, *j;
    Data tmpData;

    i = linkedList->current = linkedList->head->next;
    // Bubble Sort
    for (; i != linkedList->tail; i = i->next) {
        for (j = i->next; j != linkedList->tail; j = j->next) {
            if (dataCompare((i + 1), (j + 1)) > 0) {
                memcpy(&tmpData, i + 1, size);
                memcpy(i + 1, j + 1, size);
                memcpy(j + 1, &tmpData, size);
            }
        }
    }
}
//————————————————————————————————————
//  링크드리스트의 특정 데이터 검색
//————————————————————————————————————
Node* LinearSearchUnique(LinkedList* linkedList, void* key, int (*dataCompare)(void*, void*))
{
    linkedList->current = linkedList->head->next;
    for (; linkedList->current != linkedList->tail; linkedList->current = linkedList->current->next) {
        if (dataCompare((Node*)(linkedList->current + 1), (Data*)key) == 0)
            break;
    }
    return (linkedList->current != linkedList->tail) ? linkedList->current : NULL;
}
//————————————————————————————————————
//  링크드리스트의 중복 데이터의 주소값들을 복사
//————————————————————————————————————
void LinearSearchDuplicate(LinkedList* linkedList, Node*** indexes, int* count, void* key, int (*dataCompare)(void*, void*))
{
    int idx = 0;
    // 개수를 세기 위해 앞에서 뒤로 이동한다.
    linkedList->current = linkedList->head->next;
    for (; linkedList->current != linkedList->tail; linkedList->current = linkedList->current->next) {
        if (dataCompare((Node*)(linkedList->current + 1), (Data*)key) == 0)
            idx++; // 복사
    }
    *count = idx;
    // 데이터를 담고 있는 주소들을 위한 공간을 할당한다.
    (*indexes) = (Node**)malloc(sizeof(Node) * (*count));
    // 다시 처음으로 되돌림.
    linkedList->current = linkedList->tail->previous;
    for (idx = (*count) - 1; linkedList->current != linkedList->head; linkedList->current = linkedList->current->previous) {
        if (dataCompare((Node*)(linkedList->current + 1), (Data*)key) == 0)
            (*indexes)[idx–] = (linkedList->current);
    }
}
//————————————————————————————————————
//  링크드리스트의 특정 노드 삭제
//————————————————————————————————————
Node* Delete(LinkedList* linkedList, Node* index, void (*dataFree)(void*))
{
    Node* tmp;

    index->previous->next = index->next; // 포인터 복사(현재 노드의 Right Pointer)
    index->next->previous = index->previous; // 포인터 복사(현재 노드의 Left Pointer)

    free(index);

    (linkedList->length)–;
    return NULL;
}
//————————————————————————————————————
//  링크드리스트의 모든 데이터 초기화
//————————————————————————————————————
void Destroy(LinkedList* linkedList, void (*dataFree)(void*))
{
    Node* tmp;
    linkedList->current = linkedList->head->next; // 삭제할 시작 노드
    while (linkedList->current != linkedList->tail) // 삭제할 시작 노드
    {
        tmp = linkedList->current->next; // 다음 노드를 임시 저장

        Delete(linkedList, linkedList->current, NULL); // 노드 삭제
        linkedList->current = tmp; // 다음 삭제할 노드 정보를 복구
    }
}

//————————————————————————————————————
//  링크드리스트의 특정 데이터를 index 오른쪽으로 삽입
//————————————————————————————————————
Node* InsertAfter(LinkedList* linkedList, Node* index, void* item, size_t size)
{
    linkedList->current = CreateEmptyNode(size); // 새 노드의 동적 메모리 할당
    assert(linkedList->current != NULL); // 메모리 할당이 실패하면 종료
    memcpy(linkedList->current + 1, item, size); // 추가할 데이터를 새 노드의 데이터 영역에 저장
    // index 노드 뒤에 새 노드 연결
    linkedList->current->previous = index; // index <- [*| current |*] -> NULL
    linkedList->current->next = index->next; // index <- [*| current |*] -> index->next
    index->next->previous = linkedList->current; // 기존 index 다음 노드의 전 노드가 새 노드를 가리키게 한다
    index->next = linkedList->current; // 기존 index노드 다음 노드가 새 노드를 가리키게 한다

    (linkedList->length)++;
    return linkedList->current; // 새롭게 추가한 노드의 주소를 리턴한다.
}
//————————————————————————————————————
//  링크드리스트의 특정 데이터를 index 왼쪽으로 삽입
//————————————————————————————————————
Node* InsertBefore(LinkedList* linkedList, Node* index, void* item, size_t size)
{
    linkedList->current = CreateEmptyNode(size); // 새 노드의 동적 메모리 할당
    assert(linkedList->current != NULL); // 메모리 할당이 실패하면 종료
    memcpy(linkedList->current + 1, item, size); // 추가할 데이터를 새 노드의 데이터 영역에 저장
    // index 노드 앞에 새 노드 연결
    linkedList->current->next = index; //            NULL <- [*| current |*] -> index
    linkedList->current->previous = index->previous; // index->previous <- [*| current |*] -> index

    index->previous->next = linkedList->current; // 기존 index 전 노드의 다음 노드가 새 노드를 가리키게 한다
    index->previous = linkedList->current; // 기존 index 노드 이전 노드가 새 노드를 가리키게 한다

    (linkedList->length)++;
    return linkedList->current; // 새롭게 추가한 노드의 주소를 리턴한다.
}
//————————————————————————————————————
//  링크드리스트의 데이터 초기화
//————————————————————————————————————
void Create(LinkedList* list)
{
    // NULL<-[ LP | head | RP ]->NULL
    // ——————————————————-
    list->head = CreateEmptyNode(0);
    assert(list->head != NULL);
    list->head->previous = list->head;
    // ——————————————————-
    // NULL<-[ LP | tail | RP ]->NULL
    // ——————————————————-
    list->tail = CreateEmptyNode(0);
    assert(list->tail != NULL);

    list->tail->next = list->tail;
    // ——————————————————-

    list->head->next = list->tail; // NULL<-[ LP | head | RP ]->     <- [ LP | tail | RP ]->NULL
    list->tail->previous = list->head; // NULL<-[ LP | head | RP ]—-><—-[ LP | tail | RP ]->NULL
    list->current = NULL; // [current]->[head]
    list->length = 0;
}
int main()
{
    LinkedList linkedList;
    Data data;
    Node* index;
    Node** indexes;

    int count = 0;
    int i;
    int totalInp = 0;
    // 링크드리스트 초기화와 데이터 입력
    Create(&linkedList);
    while (1) {
        printf("나이를 입력하세요 (종료-음수) : ");
        scanf("%d", &data);
        if (data < 0)
            break; // 음수가 입력되면 종료
        totalInp++;
        AppendFromHead(&linkedList, &data, sizeof(data)); // 링크드리스트의 앞에서 추가
    }
    if (totalInp < 1) {
        printf("자료가 존재하지 않아 실행할 수 없습니다!\n");
        return 1;
    }
    printf("링크드리스트에 입력된 데이터 : ");
    Display(&linkedList, nodeDataPrint); // 입력된 데이터 출력
    printf("\n\n");

    // 최초의 데이터 검색하여 출력하기
    printf("검색할 나이를 입력하세요 : ");
    scanf("%d", &data);
    index = LinearSearchUnique(&linkedList, &data, integerCompare);
    if (index != NULL) {
        GetAt(&linkedList, index, &data, sizeof(data));
        printf("링크드리스트에서 검색한 값 : %d\n", data);
    }
    else {
        printf("원하는 데이터가 없습니다!\n");
    }
    printf("\n");
    // 동일한 데이터 모두 검색하여 출력하기
    printf("검색할 나이를 입력하세요(동일 데이터 모두 검색) : ");
    scanf("%d", &data);
    LinearSearchDuplicate(&linkedList, &indexes, &count, &data, integerCompare);
    if (count > 0) {
        printf("검색한 데이터 모두 출력하기 : ");
        for (i = 0; i < count; i++) {
            GetAt(&linkedList, indexes[i], &data, sizeof(data));
            printf("% 5d", data);
        }
        printf("\n");
    }
    else {
        printf("원하는 데이터가 없습니다!\n");
    }
    printf("\n");

    // 원하는 나이 삭제하기
    printf("삭제할 나이를 입력하세요 : ");
    scanf("%d", &data);
    index = LinearSearchUnique(&linkedList, &data, integerCompare);
    if (index != NULL) {
        Delete(&linkedList, index, NULL); // 데이터 삭제
        Display(&linkedList, nodeDataPrint); // 삭제한 후 데이터 출력
    }
    else {
        printf("원하는 데이터가 없습니다!\n");
    }
    printf("\n\n");

    // 정렬하기
    printf("정렬하여 출력 : ");
    Sort(&linkedList, sizeof(data), integerCompare);
    Display(&linkedList, nodeDataPrint); // 정렬한 후 데이터 출력
    printf("\n\n");

    // 링크드리스트 뒤에 데이터 추가
    printf("끝에 추가할 나이를 입력하세요 : ");
    scanf("%d", &data);
    AppendFromTail(&linkedList, &data, sizeof(data));
    Display(&linkedList, nodeDataPrint);
    printf("\n\n");

    // 특정 노드 뒤에 데이터 추가
    printf("어떤 값 뒤에 추가할까요 : ");
    scanf("%d", &data);
    index = LinearSearchUnique(&linkedList, &data, integerCompare);
    if (index != NULL) {
        printf("추가할 나이 입력 : ");
        scanf("%d", &data);
        InsertAfter(&linkedList, index, &data, sizeof(data));
        Display(&linkedList, nodeDataPrint); // 추가한 후 데이터 출력
        printf("\n");
    }
    else {
        printf("원하는 데이터가 없습니다!\n");
    }
    printf("\n");

    // 특정 노드 앞에 데이터 추가
    printf("어떤 값 앞에 추가할까요 : ");
    scanf("%d", &data);
    index = LinearSearchUnique(&linkedList, &data, integerCompare);
    if (index != NULL) {
        printf("추가할 나이 입력 : ");
        scanf("%d", &data);
        InsertBefore(&linkedList, index, &data, sizeof(data));
        Display(&linkedList, nodeDataPrint); // 추가한 후 데이터 출력
        printf("\n");
    }
    else {
        printf("원하는 데이터가 없습니다!\n");
    }
    printf("\n");

    // 링크드리스트 초기화
    printf("링크드리스트를 초기화합니다…");
    Destroy(&linkedList, NULL);
    Display(&linkedList, nodeDataPrint); // 초기화 후 데이터 출력
    printf("\n");
    return 0;
}
```
