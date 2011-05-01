#define do_sha_block

#1. Prepare the message schedule

#W_t(i) = M_t(i) where 0 ≤ t ≤ 15

    lw @data, 0[@chunk]         #copy the chunk into stack memory, because we're gonna be extending it
    sw @data, 0[$sp]
    
    lw @data, 4[@chunk]
    sw @data, 4[$sp]
    
    lw @data, 8[@chunk]
    sw @data, 8[$sp]
    
    lw @data, 12[@chunk]
    sw @data, 12[$sp]
    
    lw @data, 16[@chunk]
    sw @data, 16[$sp]
    
    lw @data, 20[@chunk]
    sw @data, 20[$sp]
    
    lw @data, 24[@chunk]
    sw @data, 24[$sp]
    
    lw @data, 28[@chunk]
    sw @data, 28[$sp]
    
    lw @data, 32[@chunk]
    sw @data, 32[$sp]
    
    lw @data, 36[@chunk]
    sw @data, 36[$sp]
    
    lw @data, 40[@chunk]
    sw @data, 40[$sp]
    
    lw @data, 44[@chunk]
    sw @data, 44[$sp]
    
    lw @data, 48[@chunk]
    sw @data, 48[$sp]
    
    lw @data, 52[@chunk]
    sw @data, 52[$sp]
    
    lw @data, 56[@chunk]
    sw @data, 56[$sp]
    
    lw @data, 60[@chunk]
    sw @data, 60[$sp]

@temp = $t5
@w = $t6
@i = $t7
@limit = $s0

#W_t(i) = ROTL_1(W[t−3] ⊕ W[t−8] ⊕ W[t−14] ⊕ W[t−16]) where 16 ≤ t ≤ 79

    addiu @i, $sp, 64           #16 words == 64 bytes
    addiu @limit, $sp, 320      #80 words == 320 bytes
        
extend_loop:                #extend the message schedule
    lw      @w, -12[@i]
    lw      @temp, -32[@i]
    xor     @w, @w, @temp
    lw      @temp, -56[@i]
    xor     @w, @w, @temp
    lw      @temp, -64[@i]
    xor     @w, @w, @temp
    
    srl     @temp, @w, 31		#leftrotate @w by 1
    sll     @w, @w, 1
    or      @w, @w, @temp
    sw      @w, 0[@i]
    
    addiu   @i, @i, 4
	blt		@i, @limit, extend_loop
    
# 2. Initialize the five working variables, a, b, c, d, and e, with the (i-1)st hash value:

@a = $s1
@b = $s2
@c = $s3
@d = $s4
@e = $s5

    move    @a, @h0
    move    @b, @h1
    move    @c, @h2
    move    @d, @h3
    move    @e, @h4

# 3. For t=0 to 79:
# {
# T = ROTL_5(a) + f_t(b, c, d) + e + K_t + W_t
# e = d
# d = c
# c = ROTL 30 (b )
# b = a
# a = T
# }

    li      @i, 0

confuse_loop:

#cases for computing f_t(b, c, d) + K_t
    addiu   @temp, @i, -80      # 0 ≤ i ≤ 19
    bltz    @temp, case0
    
    addiu   @temp, @i, -80      # 20 ≤ i ≤ 39
    bltz    @temp, case20
    
    addiu   @temp, @i, -80      # 40 ≤ i ≤ 59
    bltz    @temp, case40
    
    j       case60              # 60 ≤ i ≤ 79

@f = $s6
     
case0:
#Ch(b, c, d) = (b ∧ c) ⊕ ( ¬ b ∧ d) 


    and     @f, @b, @c          #@f    CONTAINS b ∧ c
    nor     @temp, @b, $0       #@temp CONTAINS ¬ b
    and     @temp, @temp, @d    #@temp CONTAINS ¬ b ∧ d
    xor     @f, @f, @temp       #@f    CONTAINS (b ∧ c) ⊕ (¬ b ∧ d)
    
    li      @temp, 0x5A827999
    addu    @f, @f, @temp
    
    j       end_case
case20:
#Parity(b, c, d) = b ⊕ c ⊕ d

    xor     @f, @b, @c
    xor     @f, @f, @d
    
    li      @temp, 0x6ED9EBA1
    addu    @f, @f, @temp

    j       end_case
case40:
#Maj(b, c, d) = (b ∧ c) ⊕ (b ∧ d) ⊕ (c ∧ d) 

    and     @f, @b, @c
    and     @temp, @b, @d
    xor     @f, @f, @temp
    
    and     @temp, @c, @d
    xor     @f, @f, @temp
   
    li      @temp, 0x8F1BBCDC
    addu    @f, @f, @temp
    
    j       end_case
case60:
#Parity(b, c, d) = b ⊕ c ⊕ d

    xor     @f, @b, @c
    xor     @f, @f, @d
    
    li      @temp, 0xCA62C1D6 
    addu    @f, @f, @temp

end_case:

    addu    @f, @f, @e              #@f CONTAINS f_t(b, c, d) + e + K_t


	#TODO: Do I really need to use $at?

    sll     @temp, @a, 5
    srl     $at, @a, 27
    or      @temp, @temp, $at       #@temp CONTAINS ROTL_5(a)
    
    addu    $at, @w, @i             #compute location of w[i]
    lw      $at, 0[$at]             #load w[i] into $at
    
    addu    @temp, @temp, $at       #@temp CONTAINS ROTL_5(a) + w[i]
    addu    @temp, @temp, @f        #add @f to @temp.  @f is now free for whatever use we need.
    
    move    @e, @d
    move    @d, @c
    
    sll     @f, @b, 30              #@f is now free, remember?
    srl     @c, @b, 2
    or      @c, @c, @f
    
    move    @b, @a
    move    @a, @temp
    
    addiu   @i, @i, 4
    
	#If i<80, we're not done yet.  Go back to the top of confuse_loop
    addiu   @temp, @i, -320         #80 words == 320 bytes
    bltz    @temp, confuse_loop
    
    addu    @h0, @h0, @a
    addu    @h1, @h1, @b
    addu    @h2, @h2, @c
    addu    @h3, @h3, @d
    addu    @h4, @h4, @e

#end

@message = $a0      #pointer to message
@hlength = $a1      #length of message in bits, high part
@llength = $a2		#length of message in bits, low part

    sll     $t0, @message, 30       #$t0 CONTAINS 0 if the message is word aligned
    sltu    $v0, $0, $t0			#If not word aligned, set error code
    bne     $v0, $0, end			#Give up if not word aligned

@blocks     = $t0
@bits       = $t1

    srl     $t0, @llength, 3
    sll     $t1, @hlength, 29
    or      $t0, $t1, $t0
    srl     $t0, $t0, 6
    sll     $t0, $t0, 6             #$t0 CONTAINS the length of the message in bytes, truncated to the lowest block

    sll     $t1, @llength, 23       
    srl     @bits, @llength, 23     #$t1 CONTAINS (number of bits) % 512

@srcitr     = $t3
@dstitr     = $t4
@dstend     = $t5
@mask       = $t6
@data       = $t7
    
    move    @srcitr, @message
    move    @dstitr,                #assign the memory location of the last block to @dstitr
    addiu   @dstend, @dstitr, 64    #there are 64 bytes in a 512 bit block

copy_loop:
    sltu    @mask, $0, @bits
    sra     @mask, @mask, @bits
    lw      @data, 0[@srcitr]
    
    sw      @data, 0[@dstitr]
    addiu   @srcitr, 4
    addiu   @dstitr, 4
    ble     @dstitr, @dstend, copy_loop
    
addiu   $sp, $sp, -320
@chunk      =

@h0         = $t0
@h1         = $t1
@h2         = $t2
@h3         = $t3
@h4         = $t4

    li @h0, 0x67452301
    li @h1, 0xEFCDAB89
    li @h2, 0x98BADCFE
    li @h3, 0x10325476
    li @h4, 0xC3D2E1F0
    
main_loop:                  #loop over message chunks
    
    
    
    
    
    
    move    @i, @sp
    addiu   @limit, @sp, 320
    
    
    


@f =
@temp =






    #We will have to copy out the data, because it is going to be extended by the padding


    srl $t0, @dlength,  5           #Divide length by 32 to get floor(dlength) in words
    sll $t0, @dlength,  2           #Multiply again by four bytes/word, because we want to compute the end address in bytes
    add $t0, @mdata,    @dlength    #$t0 CONTAINS the word-aligned address of where to add my padding.


    sll $t1, @dlength, 
    srl $t1, @dlength, 29   #By zeroing the first 29 bits, we compute @dlength % 8


    lw  $t1, @dlength, 

    #l + 65 + k = 0 mod 512