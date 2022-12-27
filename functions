using HorizonSideRobots


function get_left_down_angle!(r::Robot)::NTuple{2, Int}# перемещает робота в нижний левый угол, возвращает количество шагов
    steps_to_left_border = move_until_border!(r, West)
    steps_to_down_border = move_until_border!(r, Sud)
    return (steps_to_down_border, steps_to_left_border)
end


function get_to_origin!(r::Robot, steps_to_origin::NTuple{2, Int})
    for (i, side) in enumerate((Nord, Ost))
        moves!(r, side, steps_to_origin[i])
    end
end


function next_side(side::HorizonSide)::HorizonSide
    return HorizonSide((Int(side) + 1) % 4)
end


function inverse_side(side::HorizonSide)::HorizonSide
    inv_side = HorizonSide((Int(side) + 2) % 4)
    return inv_side
end


function along!(stop_condition::Function, robot, side) 
    while !stop_condition(side)
        move!(robot, side)
    end
end


function numsteps_along!(stop_condition, robot, side) 
    n_steps = 0
    while !stop_condition(side)
        move!(robot, side)
        n_steps += 1
    end
    return n_steps
end


function along!(stop_condition, robot, side, max_num)
    n_steps = 0
    while !stop_condition(side) && n_steps < max_num
        move!(robot, side)
        n_steps += 1
    end
    return n_steps
end


function along!(robot, side) 
    while !isborder(robot, side)
        move!(robot, side)
    end
end


function numsteps_along!(robot, side) 
    n_steps = 0
    while !isborder(robot, side)
        move!(robot, side)
        n_steps += 1
    end
    return n_steps
end

function along!(robot, side, num_steps)
    n_steps = 0
    while !isborder(robot, side) && n_steps < num_steps
        move!(robot, side)
        n_steps += 1
    end
end


function snake!(stop_condition::Function, robot, (move_side, next_row_side)::NTuple{2,HorizonSide}=(Ost,Nord))
    while !stop_condition(side) && !isborder(next_row_side)
        while !isborder(robot, move_side)
            move!(move_side)
        end

        move!(next_row_side)
        move_side = inverse_side(move_side)
    end
end


function snake!(robot, (move_side, next_row_side)::NTuple{2,HorizonSide}=(Ost,Nord))
    get_left_down_angle!(robot)
    while !isborder(next_row_side)
        while !isborder(robot, move_side)
            move!(move_side)
        end

        move!(next_row_side)
        move_side = inverse_side(move_side)
    end
end


function spiral!(stop_condition::Function, robot)
    n_steps = 1
    side = Ost
    while !stop_condition(side)
        along!(stop_condition, robot, side, n_steps)
        side = next_side(side)
        along!(stop_condition, robot, side, n_steps)
        side = next_side(side)
        n_steps +=1
    end
end

function shatl!(stop_condition::Function, robot::Robot, side::HorizonSide)
    if !stop_condition(side)
        move!(robot, side)
    end
end
