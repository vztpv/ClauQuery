# 구상중... with claude.ai

## 기본 문법 정리

### 1. 리터럴 값 비교
```
$search = {
    workspace = { /game/provinces }
    to = { /temp/result }
    cond = {
        @$ = {
            owner = "FRA"              # 정확히 일치
            development = { $gt = 20 } # 20보다 큼
            culture = { $in = ["french", "occitan"] }
        }
    }
}
```

### 2. 다른 경로의 값 참조
```
$search = {
    workspace = { /game/provinces }
    to = { /temp/result }
    cond = {
        @$ = {
            # 플레이어 소유가 아닌 것
            owner = { $ne = $ref("/game/player/tag") }
            
            # 평균보다 높은 개발도
            development = { $gt = $ref("/game/average_dev") }
        }
    }
}
```

### 3. 현재 객체의 다른 필드 참조
```
$search = {
    workspace = { /game/provinces }
    to = { /temp/result }
    cond = {
        @$ = {
            # 세금이 생산보다 높음
            tax_income = { $gt = "$this.production_income" }
            
            # 인구가 개발도의 100배보다 많음
            population = { $gt = { $calc = "$this.development * 100" } }
        }
    }
}
```

### 4. 계산식 사용
```
$search = {
    workspace = { /game/provinces }
    to = { /temp/result }
    cond = {
        @$ = {
            # 총 수입이 10 이상
            $calc = {
                expr = "$this.tax_income + $this.production_income"
                $gt = 10
            }
        }
    }
}
```

### 5. 배열 조건
```
$search = {
    workspace = { /game/provinces }
    to = { /temp/result }
    cond = {
        @$ = {
            # 태그 중 하나라도 포함
            tags = { $contains = "rich" }
            
            # 여러 개 중 하나
            terrain = { $in = ["mountain", "hills"] }
            
            # 건물이 3개 이상
            buildings = { $count = { $gte = 3 } }
        }
    }
}
```

### 6. 논리 연산
```
$search = {
    workspace = { /game/provinces }
    to = { /temp/result }
    cond = {
        @$ = {
            # AND (기본)
            owner = "FRA"
            development = { $gt = 20 }
            
            # OR
            $or = [
                { terrain = "mountain" },
                { has_fort = yes }
            ]
            
            # NOT
            $not = {
                culture = "french"
            }
        }
    }
}
```

## 실전 예시

### 예시 1: 부유한 province 찾아서 개발도 증가
```
$query = {
    $search = {
        workspace = { /game/provinces }
        to = { /temp/rich }
        cond = {
            @$ = {
                development = { $gt = 20 }
                owner = "FRA"
            }
        }
    }
    
    $update = {
        workspace = { /game/provinces }
        keys = { /temp/rich }
        @development = { $add = 5 }
        @tax_income = { $mul = 1.1 }
    }
}
```

### 예시 2: 적대국 찾기
```
$query = {
    $search = {
        workspace = { /game/countries }
        to = { /temp/enemies }
        cond = {
            @$ = {
                # 플레이어가 아님
                tag = { $ne = $ref("/game/player/tag") }
                
                # 플레이어의 라이벌
                $or = [
                    { tag = { $in = $ref("/game/player/rivals") } },
                    { rivals = { $contains = $ref("/game/player/tag") } }
                ]
                
                # 군사력이 더 약함
                military_strength = {
                    $lt = $ref("/game/player/military_strength")
                }
            }
        }
    }
}
```

### 예시 3: 평균 이상 province
```
$query = {
    $search = {
        workspace = { /game/provinces }
        to = { /temp/above_average }
        cond = {
            @$ = {
                development = {
                    $gt = {
                        $avg = "/game/provinces.*.development"
                    }
                }
            }
        }
    }
    
    $insert = {
        workspace = { /game/provinces }
        keys = { /temp/above_average }
        @special_status = "developed"
        @bonus_modifier = 1.1
    }
}
```

### 예시 4: 조건부 업데이트
```
$query = {
    $search = {
        workspace = { /game/provinces }
        to = { /temp/all }
        cond = {
            @$ = { }  # 모두 선택
        }
    }
    
    $update = {
        workspace = { /game/provinces }
        keys = { /temp/all }
        
        # 개발도에 따라 다른 보너스
        @bonus = {
            $if = {
                condition = { $calc = "$this.development > 30" }
                then = { $set = 2.0 }
                else = {
                    $if = {
                        condition = { $calc = "$this.development > 20" }
                        then = { $set = 1.5 }
                        else = { $set = 1.0 }
                    }
                }
            }
        }
    }
}
```

### 예시 5: 인접 지역 확인
```
$query = {
    $search = {
        workspace = { /game/provinces }
        to = { /temp/border }
        cond = {
            @$ = {
                # 플레이어 소유
                owner = $ref("/game/player/tag")
                
                # 인접 지역 중 적국이 있음
                adjacent_provinces = {
                    $any = {
                        $filter = {
                            array = "$this.adjacent_provinces"
                            condition = {
                                # 인접 province의 owner를 확인
                                $calc = {
                                    adjacent_owner = "$ref('/game/provinces/' + $item).owner"
                                    is_rival = "$contains($ref('/game/player/rivals'), adjacent_owner)"
                                    expr = "is_rival"
                                }
                            }
                        }
                    }
                }
            }
        }
    }
    
    $update = {
        workspace = { /game/provinces }
        keys = { /temp/border }
        @is_border_province = yes
        @defense_priority = "high"
    }
}
```

### 예시 6: 복잡한 전쟁 목표 선정
```
$query = {
    $search = {
        workspace = { /game/countries }
        to = { /temp/targets }
        cond = {
            @$ = {
                # 적대 관계
                tag = { $in = $ref("/game/player/rivals") }
                
                # 약한 군사력
                military_strength = {
                    $lt = {
                        $calc = "$ref('/game/player/military_strength') * 1.5"
                    }
                }
                
                # 동맹이 약함
                $calc = {
                    expr = "$sum($map($this.allies, '$ref(/game/countries/' + $item).military_strength'))"
                    $lt = $ref("/game/player/military_strength")
                }
            }
        }
    }
    
    $update = {
        workspace = { /game/countries }
        keys = { /temp/targets }
        
        # 전쟁 가치 점수
        @war_score = {
            $calc = {
                territory = "$count($filter('/game/provinces', { owner = $this.tag }))"
                strategic = "$count($filter('/game/provinces', { owner = $this.tag, has_port = yes })) * 10"
                difficulty = "100 / max(1, $this.military_strength)"
                expr = "territory + strategic + difficulty"
            }
        }
    }
}
```

## 핵심 규칙

1. **리터럴**: `field = value`
2. **비교**: `field = { $op = value }`
3. **참조**: `field = { $op = $ref("/path") }`
4. **계산**: `field = { $op = { $calc = "expr" } }`
5. **현재 객체**: `$this.field`
6. **조건문**: `{ $if = { condition, then, else } }`
