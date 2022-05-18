# 아이템 50. 컬렉션 처리 단계 수를 제한하라

map + filter + map → map + filterNotNull → mapNotNull 처럼 처리 단계 수를 줄이면 비용을 절약할 수 있다.
