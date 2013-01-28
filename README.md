nmap-scada
==========

nse scripts for scada identification


local http = require "http"
local nmap = require "nmap"
local shortport = require "shortport"
local strbuf = require "strbuf"
local table = require "table"

description = [[
Checks for SCADA Siemens <code>S7-3** , PCS7 </code> devices.

The higher the verbosity or debug level, the more disallowed entries are shown.
]]

---
--@output
-- 80/tcp  open   http    syn-ack
-- |  foo
-- |_ var



author = "Jose Ramon Palanco, drainware"
license = "Same as Nmap--See http://nmap.org/book/man-legal.html"
categories = {"default", "discovery", "safe"}

portrule = shortport.http
local last_len = 0


-- parse all disallowed entries in body and add them to a strbuf
local function verify_pcs7(body, output)
	local version = nil
	if string.find (body, "/S7Web.css") then
	  version = body:match("(%w+)")
	  version = "fifa 12"
	  output = "version:" .. version
	  return true
	else
	  return nil
	end -- if


	--for line in body:gmatch("[^\r\n]+") do 
	--	for w in line:gmatch('[Dd]isallow:%s*(.*)') do 
	--		w = w:gsub("%s*#.*", "")
	--		buildOutput(output, w)
	--	end
	--end
	--return #output
end

action = function(host, port)
        local verified, noun 
	local answer = http.get(host, port, "/Portal0000.htm" )

	if answer.status ~= 200 then
		return nil
	end

	local v_level = nmap.verbosity() + (nmap.debugging()*2)
	local output = strbuf.new()
	local detail = 15

	verified = verify_pcs7(answer.body, output)

	if verified == nil then 
		return
	else
	    print "------->"
	end

	-- verbose/debug mode, print 50 entries
	if v_level > 1 and v_level < 5 then 
		detail = 40 
	-- double debug mode, print everything
	elseif v_level >= 5 then
		detail = verified
	end

	-- check we have enough entries
	--if detail > dis_count then 
	--	detail = dis_count
	--end

	--noun = dis_count == 1 and "entry " or "entries "

	--local shown = (detail == 0 or detail == dis_count) 
    --             and "\n" or '(' .. detail .. ' shown)\n'

    --return "finalizado"
	return  output 
end