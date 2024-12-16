datas segment
    array db 100 dup('$')
    len dw 0
    upper_case db "ABCDEFGHIJKLMNOPQRSTUVWXYZ", 0
    lower_case db "abcdefghijklmnopqrstuvwxyz", 0
    digits db "0123456789", 0
    special_chars db "#@$%&*()_+[]{}!" ,0
    ;special_chars db "!@#$%^&*()_+[]{}|:,.<>?/~" ,0
    valid_upper_length db 26
    valid_lower_length db 26
    valid_digits_length db 10
    valid_special_length db 15
    constraints dw ""
    isUpper DB 0
    isLower DB 0
    isDigit DB 0
    isSpecial DB 0
    wrong_pass db 0Dh, 0Ah, "weak password", 0Dh, 0Ah, '$'
    wrong_mess db 0Dh, 0Ah,"please check the constraints and renter the password", 0Dh, 0Ah, '$'
    newline db 0Dh, 0Ah, '$'
    strong_Pass db  0Dh, 0Ah,"strong password",  0Dh, 0Ah,'$'
    intro_msg db "Welcome to Forti Pass!", 0Dh, 0Ah, "$"
    constraints_msg db "Password must meet the following criteria:", 0Dh, 0Ah, "$"
    constraints1 db "1. At least 12 characters.", 0Dh, 0Ah, "$"
    constraints2 db "2. Must contain uppercase letters, lowercase letters, numbers, and special characters.", 0Dh, 0Ah, "$"
    constraints3 db "3. No spaces or predictable patterns.", 0Dh, 0Ah, "$"
    menu_msg db  0Dh, 0Ah,"Choose an option:", 0Dh, 0Ah, "$"
    option1 db "1. Test your password", 0Dh, 0Ah, "$"
    option2 db "2. Generate a new password", 0Dh, 0Ah, "$"
    option3 db "3. Exit", 0Dh, 0Ah, "$"
    invalid_option db "Invalid option. Please try again.", 0Dh, 0Ah, "$"
    password_prompt db  0Dh, 0Ah, "Enter your password: $"
    gen_password_msg db "Password generation feature is available.", 0Dh, 0Ah, "$"
datas ends
codes segment
    main proc far
    assume cs:codes, ds:datas
    mov ax, datas
    mov ds, ax
    ; Display program name (only once)
    lea dx, intro_msg
    mov ah, 09h
    int 21h
    ; Display password constraints
    lea dx, constraints_msg
    mov ah, 09h
    int 21h
    lea dx, constraints1
    mov ah, 09h
    int 21h
    lea dx, constraints2
    mov ah, 09h
    int 21h
    lea dx, constraints3
    mov ah, 09h
    int 21h
    ; Start the menu options
menu:
    lea dx, menu_msg
    mov ah, 09h
    int 21h
    lea dx, option1
    mov ah, 09h
    int 21h
    lea dx, option2
    mov ah, 09h
    int 21h
    lea dx, option3
    mov ah, 09h
    int 21h
    ; Get user choice
    call readinp
    cmp al, '1'
    je test_password
    cmp al, '2'
    je generate_password
    cmp al, '3'
    je exit_program
    lea dx, invalid_option
    mov ah, 09h
    int 21h
    jmp menu
test_password:
    lea dx, password_prompt
    mov ah, 09h
    int 21h
    lea si, array
    mov cx, 0
    call read_password
    ; Check password
    call check_password
    jmp menu
generate_password:
    lea dx, gen_password_msg
    mov ah, 09h
    int 21h
    lea si, array
    mov cx, 12
    call generate_strong_password
    jmp menu
exit_program:
    mov ah, 4Ch
    int 21h

main endp
check_password proc near
    ; Reset flags before checking password
    mov isUpper, 0
    mov isLower, 0
    mov isDigit, 0
    mov isSpecial, 0
    lea si, array
    mov cx, [len] ; Use the actual length of the password
    cmp len ,12
    jl fail
check_loop:
    mov al, [si]
    cmp al, 0Dh ; Check if we reached the end of input (Enter key)
    je check_done
    cmp al, 'A'
    jl check_lowercase
    cmp al, 'Z'
    jg check_lowercase
    mov isUpper, 1
    jmp next_char

check_lowercase:
    cmp al, 'a'
    jl check_digit
    cmp al, 'z'
    jg check_digit
    mov isLower, 1
    jmp next_char

check_digit:
    cmp al, '0'
    jl check_special
    cmp al, '9'
    jg check_special
    mov isDigit, 1
    jmp next_char

check_special:
    cmp al, '!'
    jl next_char
    cmp al, '/'
    jg next_char
    mov isSpecial, 1
    jmp next_char

next_char:
    inc si
    loop check_loop

check_done:
    ; Check if all constraints are satisfied
    cmp isUpper, 0
    je fail
    cmp isLower, 0
    je fail
    cmp isDigit, 0
    je fail
    cmp isSpecial, 0
    je fail
    ; Password is strong
    lea dx, strong_Pass
    mov ah, 09h
    int 21h
    ret

fail:
    ; Password is weak
    lea dx, wrong_pass
    mov ah, 09h
    int 21h
    lea dx, wrong_mess
    mov ah, 09h
    int 21h
    ret

check_password endp

read_password proc near
    read_char:
        mov ah, 01h
        int 21h
        cmp al, 0Dh
        je done
        mov [si], al
        inc si
        inc cx
        jmp read_char
    done:
        mov [len], cx
        ret
read_password endp

readinp proc near
    mov ah, 01h
    int 21h
    ret
readinp endp

generate_strong_password proc    
    call random_uppercase
    mov ah, al       
    call display_char
    call random_special
    mov al, al     
    call display_char
    call random_uppercase2
    mov bl, al       
    call display_char
    call random_digit
    mov cl, al     
    call display_char
    call random_lowercase
    mov dl, al       
    call display_char
    call random_special2
    mov dh, al       
    call display_char
    call random_digit
    test al, al     
    call display_char 
    call random_uppercase
    mov ah, al       
    call display_char
    call random_digit2
    mov al, al     
    call display_char
    call random_uppercase3
    mov bl, al       
    call display_char
    call random_lowercase2
    mov cl, al     
    call display_char
    call random_lowercase3
    mov dl, al       
    call display_char
    call random_special3
    mov dh, al       
    call display_char
    call random_digit3
    test al, al     
    call display_char
    ret
generate_strong_password endp

random_uppercase proc   
     mov ah, 2Ch     
    int 21h              
    mov al, dl       
    and al, valid_upper_length      
     xor ah, ah
    div valid_upper_length   
    mov bx, 0
    mov bl, ah
    mov al, [upper_case + bx]
    ret
    random_uppercase endp
    random_lowercase proc  
    mov ah, 2Ch     
    int 21h           
    mov al, dl       
    and al, valid_lower_length        
    xor ah, ah
    div valid_lower_length 
    mov bx, 0
    mov bl, ah  
    mov al, [lower_case + bx]
    ret
    random_lowercase endp
    random_digit proc
     mov ah, 2Ch     
    int 21h              
    mov al, dl       
    and al, valid_digits_length         
     xor ah, ah
     div valid_digits_length     
    mov bx, 0
    mov bl, ah 
    mov al, [digits + bx]
    ret
    random_digit endp    
    random_special proc   
     mov ah, 2Ch     
    int 21h              
    mov al, dl       
    and al, valid_special_length         
    xor ah, ah
    div valid_special_length     
    mov bx, 0
    mov bl, ah  
    mov al, [special_chars + bx]
    ret
random_special endp
random_uppercase2 proc   
     mov ah, 2Ch     
    int 21h              
    mov al, dh       
    and al,valid_upper_length         
     xor ah, ah
    div valid_upper_length    
    mov bx, 0
    mov bl, ah  
    mov al, [upper_case + bx]
    ret
    random_uppercase2 endp
    random_lowercase2 proc   
     mov ah, 2Ch     
    int 21h             
    mov al, dh       
    and al, valid_lower_length      
		xor ah, ah
     div valid_lower_length    
    mov bx, 0
    mov bl, ah
    mov al, [lower_case + bx]
    ret
    random_lowercase2 endp
    random_digit2 proc   
     mov ah, 2Ch     
    int 21h              
    mov al, dh       
    and al, valid_digits_length    
     xor ah, ah
     div valid_digits_length     
    mov bx, 0
    mov bl, ah  
    mov al, [digits + bx]
    ret
    random_digit2 endp    
    random_special2 proc   
     mov ah, 2Ch     
    int 21h              
    mov al, dh       
    and al, valid_special_length      
     xor ah, ah
     div valid_special_length     
    mov bx, 0
    mov bl, ah  
    mov al, [special_chars + bx]
    ret
    random_special2 endp
    random_uppercase3 proc   
     mov ah, 2Ch     
    int 21h              
    mov al, cl       
    and al, valid_upper_length         
     xor ah, ah
    div valid_upper_length     
    mov bx, 0
    mov bl, ah  
    mov al, [upper_case + bx]
    ret
    random_uppercase3 endp
    random_lowercase3 proc   
     mov ah, 2Ch     
    int 21h              
    mov al, cl       
    and al, valid_lower_length         
     xor ah, ah
     div valid_lower_length     
    mov bx, 0
    mov bl, ah  
    mov al, [lower_case + bx]
    ret
    random_lowercase3 endp
    random_digit3 proc   
     mov ah, 2Ch     
    int 21h              
    mov al, cl       
    and al, valid_digits_length         
     xor ah, ah
     div valid_digits_length 
    mov bx, 0
    mov bl, ah  
    mov al, [digits + bx]
    ret
    random_digit3 endp    
    random_special3 proc   
     mov ah, 2Ch     
    int 21h              
    mov al, cl       
    and al, valid_special_length          
     xor ah, ah
     div valid_special_length     
    mov bx, 0
    mov bl, ah  
    mov al, [special_chars + bx]
    ret
    random_special3 endp    
display_char proc    
    mov dl, al       
    mov ah, 2        
    int 21h
    ret
display_char end
codes ends
end main
