using HorizonSideRobots
import HorizonSideRobots.move!

function inverse_side(side::HorizonSide)::HorizonSide
    inv_side = HorizonSide((Int(side) + 2) % 4)
    return inv_side
end

function moves!(r::Robot, side::HorizonSide, n_steps::Int)
    for i in 1:n_steps
        move!(r, side)
    end
end

function move_until_border!(r::Robot, side::HorizonSide)::Int
    n_steps = 0
    while !isborder(r, side)
        n_steps += 1
        move!(r, side)
    end
    return n_steps
end

function putmarkers_until_border!(r::Robot, side::HorizonSide)::Int
    n_steps = 0
    while !isborder(r, side) 
        move!(r, side)
        putmarker!(r)
        n_steps += 1
    end 
    return n_steps
end

#1-ая задач
function mark_cross!(r::Robot)
    for side in (Nord, Sud, West, Ost)
        n_steps = putmarkers_until_border!(r, side)
        moves!(r, inverse_side(side), n_steps)
    end
    putmarker!(r)
end

#2-ая задача
function mark_perimetr!(r::Robot)
    steps_to_left_down_angle = [0, 0] 
    steps_to_left_down_angle[1] = move_until_border!(r, Sud)
    steps_to_left_down_angle[2] = move_until_border!(r, West)
    for side in (Nord, Ost, Sud, West)
        putmarkers_until_border!(r, side)
    end
    moves!(r, Ost, steps_to_left_down_angle[2])
    moves!(r, Nord, steps_to_left_down_angle[1])
end


function get_left_down_angle!(r::Robot)::NTuple{2, Int}
    steps_to_left_border = move_until_border!(r, West)
    steps_to_down_border = move_until_border!(r, Sud)
    return (steps_to_down_border, steps_to_left_border)
end

function get_to_origin!(r::Robot, steps_to_origin::NTuple{2, Int})
    for (i, side) in enumerate((Nord, Ost))
        moves!(r, side, steps_to_origin[i])
    end
end
#3-ая задача
function mark_fild!(r::Robot)
    steps_to_origin = get_left_down_angle!(r)
    putmarker!(r)
    while !isborder(r, Ost)
        putmarkers_until_border!(r,Nord)
        move!(r, Ost)
        putmarker!(r)
        putmarkers_until_border!(r, Sud)
    end
    get_left_down_angle!(r)
    get_to_origin!(r, steps_to_origin)
end


function putmarkers_until_border!(r::Robot, sides::NTuple{2, HorizonSide})::Int
    n_steps = 0
    while !isborder(r, sides[1]) && !isborder(r, sides[2])
        n_steps += 1
        move!(r, sides)
        putmarker!(r)
    end
    return n_steps
end

function moves!(r::Robot, sides::NTuple{2, HorizonSide}, n_steps::Int)
    for _ in 1:n_steps
        move!(r, sides)
    end
end

function move!(r::Robot, sides::NTuple{2, HorizonSide})
    for side in sides
        move!(r, side)
    end
end

function inverse_side(sides::NTuple{2, HorizonSide})
    new_sides = (inverse_side(sides[1]), inverse_side(sides[2]))
    return new_sides
end

#4-ая задача
function X_mark_the_spot!(r::Robot)
    sides = (Nord, Ost, Sud, West)
    for i in 1:4
        first_side = sides[i]
        second_side = sides[i % 4 + 1]
        direction = (first_side, second_side)
        n_steps = putmarkers_until_border!(r, direction)
        moves!(r, inverse_side(direction), n_steps)
    end
    putmarker!(r)
end

#5-ая задача
function get_left_down_angle_modified!(r::Robot)::Vector{Tuple{HorizonSide, Int}}
    steps = []
    while !(isborder(r, West) && isborder(r, Sud))
        steps_to_West = move_until_border!(r, West)
        steps_to_Sud = move_until_border!(r, Sud)
        push!(steps, (West, steps_to_West))
        push!(steps, (Sud, steps_to_Sud))
    end
    return steps
end

function move_if_possible!(r::Robot, side::HorizonSide)::Bool
    if !isborder(r, side)
        move!(r, side)
        return true
    end
    return false
end

function inversed_path(path::Vector{Tuple{HorizonSide, Int}})::Vector{Tuple{HorizonSide, Int}}
    inv_path = []
    for step in path
        inv_step = (inverse_side(step[1]), step[2])
        push!(inv_path, inv_step)
    end
    reverse!(inv_path)
    return inv_path
end

function make_way!(r::Robot, path::Vector{Tuple{HorizonSide, Int}})
    for step in path
        moves!(r, step[1], step[2])
    end
end

function make_way_back!(r::Robot, path::Vector{Tuple{HorizonSide, Int}})
    inv_path = inversed_path(path)
    make_way!(r, inv_path)
end

function mark_inner_rectangle!(r::Robot)
    steps = get_left_down_angle_modified!(r)

    while isborder(r, Sud) && !isborder(r, Ost)
        move_until_border!(r, Nord)
        move!(r, Ost)
        while !isborder(r, Ost) && move_if_possible!(r, Sud) end
    end

    for sides in [(Sud, Ost), (Ost, Nord), (Nord, West), (West, Sud)]
        side_to_move, side_to_border = sides
        while isborder(r, side_to_border)
            putmarker!(r)
            move!(r, side_to_move)
        end
        putmarker!(r)
        move!(r, side_to_border)
    end

    get_left_down_angle_modified!(r)
    make_way_back!(r, steps)
end

#6-ая задача
#Пункта а
function mark_perimetr_with_inner_border!(r::Robot) 
    path = get_left_down_angle_modified!(r)
    mark_perimetr!(r)
    make_way_back!(r, path)
end

function moves_if_possible!(r::Robot, side::HorizonSide, n_steps::Int)::Bool
    
    while n_steps > 0 && move_if_possible!(r, side)
        n_steps -= 1
    end

    if n_steps == 0
        return true
    end

    return false
end

#Пункт б
function mark_four_cells!(r::Robot)
    path = get_left_down_angle_modified!(r)
    n_steps_to_sud = 0
    n_steps_to_west = 0
    for step in path
        if step[1] == Sud
            n_steps_to_sud += step[2]
        else
            n_steps_to_west += step[2]
        end
    end

    moves!(r, Ost, n_steps_to_west)
    putmarker!(r)
    move_until_border!(r, Ost)
    moves!(r, Nord, n_steps_to_sud)
    putmarker!(r)
    get_left_down_angle_modified!(r)

    moves!(r, Nord, n_steps_to_sud)
    putmarker!(r)
    move_until_border!(r, Nord)
    moves!(r, Ost, n_steps_to_west)
    putmarker!(r)
    get_left_down_angle_modified!(r)

    make_way_back!(r, path)
end

#7-ая задача

function find_space!(r::Robot, side::HorizonSide)
    n_steps = 1
    ort_side = HorizonSide((Int(side) + 1) % 4)
    while isborder(r, side)
        moves!(r, ort_side, n_steps)
        n_steps += 1
        ort_side = inverse_side(ort_side)
    end
end

function move_through!(r::Robot, side::HorizonSide)
    find_space!(r, side)
    move!(r, side)
end

#8-ая задача

function move_if_not_marker!(r::Robot, side::HorizonSide)::Bool
    
    if !ismarker(r)
        move!(r, side)
        return false
    end

    return true
end

function moves_if_not_marker!(r::Robot, side::HorizonSide, n_steps::Int)::Bool

    for _ in 1:n_steps
        if move_if_not_marker!(r, side)
            return true
        end
    end
    
    return false
end

function next_side(side::HorizonSide)::HorizonSide
    return HorizonSide( (Int(side) + 1 ) % 4 )
end


function move_snake_until_marker!(r::Robot)
    n_steps = 1
    cur_side = Ost
    counter = 1
    while true

        if moves_if_not_marker!(r, cur_side, n_steps)
            return
        end 

        cur_side = next_side(cur_side)

        if counter % 2 == 0
            n_steps += 1
        end

        counter += 1
    end
end

#9-ая задача


function mark_chess!(r::Robot)
    
    steps = get_left_down_angle!(r)
    to_mark = (steps[1] + steps[2]) % 2 == 0
    steps_to_ost_border = move_until_border!(r, Ost)
    move_until_border!(r, West)
    last_side = steps_to_ost_border % 2 == 1 ? Sud : Nord

    side = Nord

    while !isborder(r, Ost)
        
        while !isborder(r, side)
            if to_mark
                putmarker!(r)
            end

            move!(r, side)
            to_mark = !to_mark
        end

        if to_mark
            putmarker!(r)
        end

        move!(r, Ost)
        to_mark = !to_mark
        
        side = inverse_side(side)
    end

    while !isborder(r, last_side)
        
        while !isborder(r, side)
            if to_mark
                putmarker!(r)
            end

            move!(r, side)
            to_mark = !to_mark
        end

        if to_mark
            putmarker!(r)
        end

    end

    get_left_down_angle!(r)
    get_to_origin!(r, steps)
end

#10-ая задача

function mark_square!(r::Robot, n::Int)
    
    counter1 = 1
    counter2 = 1

    while counter1 <= n && !isborder(r, Ost)

        while counter2 < n && !isborder(r, Nord)
            putmarker!(r)
            move!(r, Nord)
            counter2 += 1
        end

        putmarker!(r)
        moves!(r, Sud, counter2 - 1)
        counter2 = 1

        move!(r, Ost)
        counter1 += 1
    end

    if isborder(r, Ost) && counter1 <= n
        while counter2 < n && !isborder(r, Nord)
            putmarker!(r)
            move!(r, Nord)
            counter2 += 1
        end

        putmarker!(r)
        moves!(r, Sud, counter2 - 1)
    end

    moves!(r, West, counter1 - 1)
end

function moves_if_possible_numeric!(r::Robot, side::HorizonSide, n_steps::Int)::Int
    
    while n_steps > 0 && move_if_possible!(r, side)
        n_steps -= 1
    end

    return n_steps
end
function mark_chess!(r::Robot, n::Int)
    
    steps = get_left_down_angle!(r)
    side = Nord
    to_mark = true

    steps_to_ost_border = move_until_border!(r, Ost)
    move_until_border!(r, West)
    last_side = steps_to_ost_border % 2 == 1 ? Sud : Nord
    last_n_steps = 0

    while !isborder(r, Ost)
        while !isborder(r, side)
            if to_mark
                mark_square!(r, n)
            end

            last_n_steps = moves_if_possible_numeric!(r, side, n)
            if last_n_steps == 0 && !isborder(r, side)
                to_mark = !to_mark
            end
        end

        if to_mark
            mark_square!(r, n)
        end

        to_mark = !to_mark

        moves_if_possible!(r, Ost, n)
        moves!(r,inverse_side(side), n - last_n_steps)
        side = inverse_side(side)
    end

end
