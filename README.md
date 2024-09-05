# 구상중...
    $query = {
    		workspace = { /data }
    $insert = {
        @x = 15
        @y = {
            z = 0
        } 
        @"a" = 3
    
        @provinces = {
            -1 = {
                x = 0
            }
            -2 = {
                x = 1
            }
        }
    }
    
    $insert = {
        x = 15
    
        provinces = {
            $ = {
                x = 0
                @y = wow2
            }
        }
    }
    
    $read = {
        x = 15 # condition
        provinces = {
            @$%key_only%also_value = { # target
                y = wow2
            }
        }
    }
    
    $read = {
        x = 15 # condition
        provinces = {
            $ = { 
                y = wow2 # condititon
                @x = %any # always read x`s value? ,
                            # %also_key ? - include key also?
            }
        }
    }
    
    <stack>
    _VALUE [ -1, -2, -4, -6 ] # ?
    MAKE_SET set_0 (ARRAY_SET OR HASH_SET) 
                            # only for primitive type?
    # chk) Merge two SET?
    
    $update = { # Parameter?
        #@x = 2 # @ : target, 2 : set value
        "a" = 3 # condition
        y = {
            @z = 4 # @ : target.
        }
        provinces = {
            $%always_true%all%just_one_more = {
                x = 0
                @y = %event_test3 # %event_test2%'x = /./x' <- support?
            }
        }
    }
    		
    $delete = {
        @x = 1 # @ : remove object., if value is 1 then remove
        "a" = 3 # condition.
        y = {
            @z = %any # %any : condition - always.
        }
        provinces = {
            @$ = { # $ : all usertype( array or object or mixed )
                x = 1 # condition.
            }
        }
    }
    	}
    	
    $search = { # read?
         workspace = { /Test/eu4/provinces }
         to = { /output }
         cond = {
             @$ = {
                 is_city = yes
                 owner = "DAN"
             }
         }
    }
# 구상중...

        # $Query -> set of functions..
        
        #  Query_${No} <- func name?
        # workspace { /data } <- cd( data ) // . <- now, .. <- parent, root <- root
        # $insert -> Query_${No}_0
        # x = 10 # condition..
        # x = 5 y = 3 # or 
        # line by? # and
        #1줄에 여러개 - or, 줄마다 and
        # 위에서 아래로..
        # iterator? - uint64_t find(key); set_idx, get_value ?
        # @x = 15 # @ : target
        #
        
        #workspace = { /data }
        _Value data
        CD 1 # 2
        # $insert = { @x = 15 @y = { z = 0 } }
        MAKE_OBJECT TEMP_0
        _Value x # key
        _Value 15
        PUSH_JSON_ELEMENT TEMP_0
        _Value y
        _Value { z = 0 } # check..
        PUSH_JSON_ELEMENT TEMP_0
        INSERT_CHILD_OF TEMP_0 
        EXIT
        
        # $insert = { @x = 15 y = { z = 0 } }
        MAKE_OBJECT TEMP_1
        _Value x
        _Value 15
        PUSH_JSON_ELEMENT TEMP_1
        _Value y
        FIND_IDX_BY_KEY
        SET_IDX
        ENTER  # with FIND_IDX_BY_KEY and SET_IDX 
        _Value z # key
        FIND_BY_KEY 
        _Value 0
        EQ 
        SET_CONDITION
        CONDITION_NOT_ZERO_GOTO +3
        INSERT_CHILD_OF TEMP_1
        EXIT
        QUIT
        INSERT_CHILD_OF TEMP_1
        EXIT
        
        # $update = { a = 3 provinces = { $ = { x = 0 @y = %event_test } } }
        MAKE_OBJECT TEMP_2
        _Value a
        FIND_BY_KEY 
        _Value 3
        EQ 
        SET_CONDITION
        CONDITION_NOT_ZERO_GOTO +3  
        UPDATE_FROM_CHILD_OF TEMP2
        EXIT
        _Value provinces
        FIND_IDX_BY_KEY
        SET_IDX
        ENTER # check
        ITERATE FUNC_TEMP_0
        QUIT
        CONDITION_ZERO_GOTO +3
        UPDATE_FROM_CHILD_OF TEMP2
        EXIT
        # end
        
        UPDATE_FROM_CHILD_OF TEMP2
        EXIT
        
        # FUNC_TEMP_0
        MAKE_OBJECT TEMP_3
        _Value x
        FIND_BY_KEY
        _Value 0
        EQ
        SET_CONDITION
        CONDITION_NOT_ZERO_GOTO +3
        UPDATE_FROM_CHILD_OF TEMP_3
        EXIT 
        _Value y 
        #NOT_EXIST_EXIT  # think case - y is not exist! 
        _Value %event_test
        CALL 0 # number of argument??
        RETURN_VALUE
        PUSH_JSON_ELEMENT TEMP_3 
        UPDATE_FROM_CHILD_OF TEMP_3
        EXIT 
        
        #
        DEBUG # : print error log?
        
        
        # 1. 조건들은 따로 먼저오겠끔? 처리한다?
        # object = { a = 5 @x = 6 }
        #-> object = { a = 5 } object = { @x = 6 }
        # 2. 분리한다? 
        
        # 자식이 empty?이면 지운다..?
        
        # read, insert, update, delete. 
           # + 사용자정의함수?
        
        
        # key = value
        "공격력" = 100
        
        # object { key = value ... }
        "타마린느" = {
            "공격력" = 50
            "방어력" = 100
            "체력" = 1000
        }
        
        # array [ value ... ]
        "영웅" = [
            "타마린느"
            "브리그" 
            "이세리아"  
        ]
        
        # load json file
        # cd? goto? (query를 할 장소로 이동)
        # query - insert(create?), read(?), update, delete
        # { json = { } global_variable = { } }
            #  load,         make_global_var(?)
        # array as arary vs array as (hash)set
        # object as array(?) vs array as (hash)map
        # goto1 - json.
        # goto2 - global_variable.
        $make_global_var = { test_var }
        $goto = { json = %root }
        # //$goto = { global_var = %none } # global_var is not used. ?
        # %is_in%global_test_var
        # %is_not_in%global_test_var
        # %value_in%in_glboal_test_var2
        # %add_to%test_var ?
        # %add_to%test_var2 ?
        # 
        $read = {
            "a" = 3
            provinces = {
                @$%event_testA = {
                    is_city = yes
                    owner = @%event_testB%str # @%add_to_key%test_var%str
                }   
            }
        }
        #
        Event = {
            id = testA
            
            # %element_id <- key(in object) or idx(in array)
            %global_var%test_var = %element_id 
        }
        
        Event = {
            id = testB
            
            %element_value%add_to_key%test_var2
            %global_var%test_var%add_to_value%test_var2
            
            # %1%3%$add # 1+3 -> 4   %input1 %input2 %$func_name
        }
        

